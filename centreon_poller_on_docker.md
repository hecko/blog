# Notes for Centreon Poller on Docker deployment

  * Uses Centreon 19.04
  * manually written Dockerfile for centreon poller
  * one central centreon central server with web interface - not running on docker yet
  * all pollers (about 60 now) run in Docker containers
  * two main docker containers:
    * centreon-engine
    * centreon-vmware - with centreon vmware daemon
  * centreon-engine container runs several services using supervisord:
    * centreon-engine
    * ssh for updating centreon config
    * postfix3 for sending emails
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