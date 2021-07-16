# JENKINS MASTER/SLAVE ON DOCKER FOR DevOps PROJECT

1. DISCLAIMER:

   a) This project is solely conducted by myself as I didn't test the     security side of it. So, I won't be held accountable for any issues you might enconter even though I will walk you through the process of setting up an architecture based on Jenkins Master/Slave over Docker.
   
   b) I won't go over the details of Docker commands or Linux and Jenkins as I assume that you are already familiar to those terms.

   c) Don't forget to talk to your manager or your team members before implementing such project so they can fully understand the pros and cons.

   d) This may fail depending on various factors:
      - bad configurations
      - The time this documentation was written
      - The platforms
   e) I'm doing this on the same system where my Jenkins Master sits.

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

     Client: Docker Engine - Community
     Version:           20.10.7
     API version:       1.41
     Go version:        go1.13.15
     Git commit:        f0df350
     Built:             Wed Jun  2 11:56:38 2021
     OS/Arch:           linux/amd64
     Context:           default
     Experimental:      true
     Server: 
   ```
2. Now configure Docker Engine to enable the API version 1.41 (My case) by following this link: https://gist.github.com/styblope/dc55e0ad2a9848f2cc3307d4819d819f

3. Once you're done type the following command:
```sh
   # curl http://your_ip_address:2375/version

   {"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"20.10.7","Details":{"ApiVersion":"1.41","Arch":"amd64","BuildTime":"2021-06-02T11:54:50.000000000+00:00","Experimental":"false","GitCommit":"b0f5bc3","GoVersion":"go1.13.15","KernelVersion":"5.8.0-59-generic","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.4.6","Details":{"GitCommit":"d71fcd7d8303cbf684402823e425e9dd2e99285d"}},{"Name":"runc","Version":"1.0.0-rc95","Details":{"GitCommit":"b9ee9c6314599f1b4a7f497e1f1f856fe433d3b7"}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":"de40ad0"}}],"Version":"20.10.7","ApiVersion":"1.41","MinAPIVersion":"1.12","GitCommit":"b0f5bc3","GoVersion":"go1.13.15","Os":"linux","Arch":"amd64","KernelVersion":"5.8.0-59-generic","BuildTime":"2021-06-02T11:54:50.000000000+00:00"}
```
4. If you've got the output above, it means you are ready to set up your master/slave architecture. But we still have couple of more settings to do.

5. Pull and run the latest images of jenkins (jenkins/jenkins:lts-jdk11) form: 
   - Dockehub: https://hub.docker.com/r/jenkins/jenkins (pull the one with lts-jdk11)
   ```sh
      # docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
   
      * Running it with a volume will allow you to retrieve your work after a disaster.
      * Running it without the detach more allow you to see the jenkins master initialAdminPassword and its flow.
   ```
   - Documentation: https://github.com/jenkinsci/docker/blob/master/README.md

6. We now need a container from the alpine/socat image (https://hub.docker.com/r/alpine/socat/) to expose the Docker Engine API. Take the first example and read the **WARNING**
```sh
   # docker pull alpine/socat
   # docker run -d --restart=always \
    -p 127.0.0.1:2376:2375 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    alpine/socat \
    tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
```
7. Open your Jenkins GUI through your http://localhost:8080 if Jenkins is open on the port 8080 and install the necessary plugings and also change your Admin Password.

8. On your Master Jenkins dashboard go to:
*Manage Jenkins > Manage Plugins > Available* search for **Docker** plugin and install it and restart jenkins.

9. You will now be able to connect your Jenkins master with any Docker running jenkins slave as you can see there's an Uncategorized session that just appears in Manage Jenkins right at the bottom of the list with the Docker logo. *Oufff! Almost there*.

10. Now go to:
  *Manage Nodes and Clouds > Configure Clouds > Add a new cloud > Docker*
  
  - Click on **Docker Cloud details** and here's where we need to provide the Docker Engine API address: 
  ```sh 
     tcp://your_Linux_ip_address:2375
     Click on *Test Connection* at the bottom right and you should see something like this: Version = 20.10.7, API Version = 1.41
  ```
  - Check the *Enabled* box

11. Click on **Docker Agent templates > Add Docker Template**
    - Labels: docker-jnlp --> choose any name then check the Enabled box
    - Name: same as Labels for me
    - Docker Image: jenkins/jnlp-slave as I'm going to use the *Attach Docker container* as my *connect method*
    - Remote File System Root: /home/jenkins as returned by the command
    ```sh
    docker inspect jenkins/jnlp-slave | grep -I WorkingDir
    ```
    - Leave the rest by default, apply and save

12. Return to create a new Build project from the main Jenkins Dashboard
    - Click on *New Item*
    - Check *Restrict where this project can be run*
    - Provide your Label Expression you configure: docker-jnlp (mine was this one)
    - Click on Execute a shell command, save annd run the build
