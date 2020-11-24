# Notes for Centreon Poller on Docker deployment

  * Uses Centreon 19.04
  * manually written Dockerfile for centreon poller
  * one central centreon server with web interface - not running on docker yet
  * one MySQL instance for centreon database on separate system
  * all pollers (about 60 now) run in Docker containers
  * two main docker containers:
    * centreon-engine
    * centreon-vmware - with centreon vmware daemon (Dockerfile build on top of https://hub.docker.com/r/moirelabs/centreon_vmware)
  * centreon-engine container runs several services using supervisord:
    * centreon-engine
    * ssh for updating centreon config
    * postfix3 for sending emails
  * centreon-vmware starts, then pulls a small git repository with vmware configuration files and depending on the hostname deploys the configuration
    * it is required to reinitialize the vmware docker container if we want to update the configuration, but this is a very cheap operation so OK
  * centreon containers are running on top od CentOS 7 and are also based on centos7 Docker layer
  * additional docker network created between the two containers (centreon-engine and centreon-vmware)
  * containers and all host Centos 7 services are managed from one central server using Ansible
  * ansible deployment scripts are run from Jenkins
  * Jenkins is also responsible for building new docker images and pushing to private docker repository
  * all configration files for ansible, docker images, everything is stored in one rather large git repository
  * Jenkins pulls the repository and runs docker build scripts from there
  * for deployment Jenkins pulls the git repository and runs ansible deployment scripts for one or more docker host systems
  * Dockerfiles combine centreon installation from RPM repos with local plugins and configuration and builds centreon-engine docker image
  * docker hosts (pollers) only run docker daemon, snmp (for monitoring) and ssh
  
## Details - building of centreon-engine images

Operator connects to Jenkins and runs jenkins operation. Operation pulls the local git repository with all configuration files and plugins for centreon poller. Runs docker_build.sh script from git repository which pulls centos:7 and installs centreon-engine-poller packages from official centreon RPM repositories. Also adds local plugins and configuration. After the build the new docker image is pushed to local docker repository.

## Details - deployment of centreon-engine image on host

Operator connects to Jenkins UI, selects "deploy centreon poller" and fills out the hostname of the host system where to deploy centreon poller. Jenkins pulls the git repository and plays and Ansible playlist to deploy docker, rpms, snmp and other bits on the docker host. Then plays another playlist to deploy centreon-engine on the docker host. When the centreon-engine docker image starts it automatically connects to the central server with own little python client and downloads the poller configuration from the central server using ssh. Then starts an actual docker-engine. The docker-engine configuration and persistence files are mounted from the main host filesystem - they are persistent during docker engine restarts - otherwise we would be waiting waaay too long beteen each image restart in centreon PENDING state for services - also we would be sending way too many non-relevant emails. Make the retention files persistent!

## Details - monitoring of docker hosts and docker containers

Docker daemons on all hosts are queried by script on central management server using the docker API. The docker API gives information about the local version of the installed and running centreon engine container - this is evaluated and if the image is too old than the newest available image in local docker repository then its re-deployed automatically calling the deployment procedure in Jenkins.

Docker centreon engine images are also re-build automatically once there is a new version of centreon engine (eg. bump from 19.04.1 to 19.04.5)
