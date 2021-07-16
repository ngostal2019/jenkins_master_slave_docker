# JENKINS MASTER/SLAVE ON DOCKER FOR DevOps PROJECT

1. DISCLAMER:

   a) This project is solely conducted by myself as I didn't test the     security side of it. So, I won't be held accountable for any issues you might enconter even though I will walk you through the process of setting up an architecture based on Jenkins Master/Slave over Docker.
   
   b) I won't go over the details of Docker commands or Linux and Jenkins as I assume that you are already familiar to those terms.

   c) Don't forget to talk to your manager or your team members before implementing such project so they can fully understand the pros and cons.

   d) This may fail depending on various factors:
      - bad configurations
      - The time this documentation was written
      - The platforms

2. Pre-requisites:

 - A Linux machine on-premise ( Ubuntu 20.04 LTS my case) or over the cloud (I didn't try this one yet)
 - The latest version of Docker for you Linux platform
 - Docker images pulled from https://hub.docker.com
   - jenkins/jenkins
   - alpine/socat
   - jenkins/jnlp-slave (Not required but will be used)

## CONFIGURATIONS

1. Install Docker Engine and make sure it was successfully installed: https://docs.docker.com/engine/install/
   ```sh
   # docker version

output:
 Client: Docker Engine - Community
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        f0df350
 Built:             Wed Jun  2 11:56:38 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

etc ...
   ```





