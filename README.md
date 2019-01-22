## Compiling Your Own Docker Binary from Source 
# Problem 
You would like to develop the Docker software and build your own Docker binary. 


## Tested Infrastructure

<table class="tg">
  <tr>
    <th class="tg-yw4l"><b>Platform</b></th>
    <th class="tg-yw4l"><b>Number of Instance</b></th>
    <th class="tg-yw4l"><b>Reading Time</b></th>
    
  </tr>
  <tr>
    <td class="tg-yw4l"><b> Play with Docker</b></td>
    <td class="tg-yw4l"><b>1</b></td>
    <td class="tg-yw4l"><b>30 min</b></td>
    
  </tr>
  
</table>

## Pre-requisite

- Create an account with [DockerHub](https://hub.docker.com)
- Open [PWD](https://labs.play-with-docker.com/) Platform on your browser 
- Click on **Add New Instance** on the left side of the screen to bring up Alpine OS instance on the right side

## Run multiple containers in an interactive and detached mode. 



# Solution 



# using just one commnad fork the docker repository from github as shown below :

```
$ git clone https://github.com/docker/docker.git
```
## you will get on screen something like this 
```
Cloning into 'docker'...
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 264474 (delta 0), reused 0 (delta 0), pack-reused 264469
Receiving objects: 100% (264474/264474), 136.92 MiB | 17.19 MiB/s, done.
Resolving deltas: 100% (179083/179083), done.
root@55eace03cbea:/# cd docker
root@55eace03cbea:/docker# ls
AUTHORS             Makefile      client         errdefs          opts       restartmanager
CHANGELOG.md        NOTICE        cmd            hack             pkg        runconfig
CONTRIBUTING.md     README.md     codecov.yml    image            plugin     vendor
Dockerfile          ROADMAP.md    container      integration      poule.yml  vendor.conf
Dockerfile.e2e      TESTING.md    contrib        integration-cli  profiles   volume
Dockerfile.simple   VENDORING.md  daemon         internal         project
Dockerfile.windows  api           distribution   layer            reference
LICENSE             builder       dockerversion  libcontainerd    registry
MAINTAINERS         cli           docs           oci              reports
root@55eace03cbea:/docker#
```
Docker is built within a Docker container. In a Docker host, you can clone the Docker repository and use the Makefile rules to build a new binary. This binary is obtained by running a privileged Docker container. The Makefile contains several targets, including a binary target:
```
cat Makefile 
```
```
# as root (or prepend with sudo)
make build 

// default: binary

// all: build
 //  $(DOCKER_RUN_DOCKER) hack/make.sh

# this takes a while and prints the output from "Docker build ." 
# It ends with: "Successfully built ..".

# as root (or prepend with sudo)
make binary

// binary: build
 // $(DOCKER_RUN_DOCKER) hack/make.sh binary

# This runs the image we created with "make build". 
# Ends with "Created binary: bundles/1.8.0-dev/binary/docker-1.8.0-dev",
# where "1.8.0-dev" is the version I'm building, 
# the next unreleased version at the time of writing.
```

## Therefore, it is as easy as sudo make binary: 

Tips :  The hack directory in the root of the Docker repository has been moved to the project directory. Therefore, the make.sh script is in fact at project/make.sh. It uses scripts for each bundle that are stored in the project/make/ directory. 
  ```
// binary: build
 // $(DOCKER_RUN_DOCKER) hack/make.sh binary
# This runs the image we created with "make build". 
# Ends with "Created binary: bundles/1.8.0-dev/binary/docker-1.8.0-dev",
# where "1.8.0-dev" is the version I'm building, 
# the next unreleased version at the time of writing.
      # as root (or prepend with sudo)
     make binary
    docker run --rm -it --privileged \
                        -e BUILDFLAGS -e DOCKER_CLIENTONLY -e DOCKER_EXECDRIVER \
                        -e DOCKER_GRAPHDRIVER -e TESTDIRS -e TESTFLAGS \
                        -e TIMEOUT \
                        -v "/tmp/docker/bundles:/go/src/github.com/docker/docker/\
                                                bundles" \
                        "docker:master" hack/make.sh binary
    ---> Making bundle: binary (in bundles/1.9.0.-dev/binary)
    Created binary: \
    /go/src/github.com/docker/docker/bundles/1.9.0-dev/binary/docker-1.9.0-dev
```

## it will take more time ..........until get something on screen like this 

You see that the binary target of the Makefile will launch a privileged Docker con‐tainer from the docker:master image, with a set of environment variables, a volume mount, and a call to the hack/make.sh binary command. 
With the current state of Docker development, the new binary will be located in the bundles/1.9.0-dev/binary/ directory. The version number might differ, depending on the state of Docker releases. 



## to make this easy 
To ease this process, you can clone the repository that accompanies this cookbook. A Vagrantfile is provided that starts an Ubuntu 14.04 virtual machine, installs the latest stable Docker release, and clones the Docker repository: 
   
   ```
   $ git clone https://github.com/how2dock/docbook
    $ cd docbook/ch04/compile/
    $ vagrant up
   ```
 Once the machine is up, ssh to it and go to the /tmp/docker directory, which should have been created during the Vagrant provisioning process. Then run make. The first time you run the Makefile, the stable Docker version installed on the machine will pull the base image being used by the Docker build process ubuntu:14.04, and then build the docker:master image defined in /tmp/docker/Dockerfile. This can take a bit of time the first time you do it: 
   
    ```
   $ vagrant ssh
    $ cd /tmp/docker
    $ sudo make binary
    docker build -t "docker:master" .
    Sending build context to Docker daemon 55.95 MB
    Sending build context to Docker daemon
    Step 0 : FROM ubuntu:14.04
    ..........
    ....
    ```
## Once this completes, you will have a new Docker binary: 
   ```
   $ cd bundles/1.9.0-dev/binary/docker
    $ ls
    docker  docker-1.9.0-dev  docker-1.9.0-dev.md5  docker-1.9.0-dev.sha256
   ```

