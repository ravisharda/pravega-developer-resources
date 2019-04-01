# Docker Commands and Instructions

## Commands

Note: See more/detailed information at [Docker CLI Commandline Documentation](https://docs.docker.com/engine/reference/commandline/docker/). 

### Images-Related
* Viewing container images: `docker images`
* Listing dangling images: `docker images --quiet --filter "dangling=true"`
* Removing container images: 
  * `docker rmi <image id>`
  * Remove forcefully (some have a dependency and don't get removed otherwise): `docker rmi -f <image id>`
* Removing all images: `docker rmi $(docker images -q)`  
* Removing dangling images: 
    ```
    # This one worked for me in Windows cmd
    FOR /f "tokens=*" %i IN ('docker images -q -f "dangling=true"') DO docker rmi %i 
    
    # This should work too (in Powershell), but it didn't. 
    docker rmi $(docker images -qf dangling=true)
    ```
* Searching for images from Microsoft: `docker search microsoft`
* Viewing container logs" `docker logs <container name/ID>`
* Copying image from one server to another:
  ```
  # Step 1: Save the docker image as a tar file on the source server. 
  # Usage: docker save -o <file path to image tar> <image name>
  docker save -o c:\aspnetblogapplication_db.tar aspnetblogapplication_db
  
  # Step 2: Copy the tar file to the destination server.
  
  # Step 3: Load the tar file into an image on the destination server.
  # Usage: docker load -i <path to image tar file> 
  docker load -i c:\aspnetblogapplication_db.tar
  ```

### Containers-Related 

* Viewing containers
  * All running containers: `docker ps`
  * All containers: `docker ps -a`
  * Formatting the output: 
    ```
    docker ps --format "{{.ID}}: {{.Command}} | {{.Ports}} | {{.Names}}"
    ```
    See more options in the [documentation](https://docs.docker.com/engine/reference/commandline/ps/) > formatting).
* Stopping a container: `docker stop <container id>`
* Stopping all containers:
  ```
  docker kill $(docker ps -q)
  or,
  sudo docker kill $(sudo docker ps -q)
  ```
* Removing a container: `docker rm <container id>`
* Deleting all stopped containers: `docker rm -vf $(docker ps -qa)`
  ```
  docker rm $(docker ps -a -q)
  sudo docker rm $(sudo docker ps -a -q)
  ```
* Running a command inside a container: `docker exec -it <container name> <command>` (e.g., `docker exec -it mongo_01 powershell`)
* Inspecting a container: `docker inspect <container id>`
* Finding the IPv4 address of the container:  `docker inspect --format '{{ .NetworkSettings.Networks.nat.IPAddress }}' <Container ID>`, where Container ID is the tag or ID of the container. 

### Host and Daemon Management
* Viewing Docker version: `docker --version` (long-hand) or `docker -v` (short-hand)
* Viewing system-wide info (such as running/paused/stopped containers, OS system info, etc.): `docker info`
* Logging in to Docker hub: `docker login --username <username> --password <password>`
* Starting and stopping Docker service (daemon):
  * Using Powershell: `Start-Service docker` and `Stop-Service docker`
* Networking
  * Listing networks (on a Docker host): 
    * `docker network ls` 
    * Powershell command: `Get-ContainerNetwork` (provides you additional information such as Subnets)
  * Removing a network: `docker network rm <network name>` (e.g., `docker network rm nat`)
  * Finding more information about a network: `docker network inspect <network name>`
 

## Quick Reference: Dockerfile instructions

Note:
* Read [Dockerfile reference](https://docs.docker.com/engine/reference/builder/) for detailed information about the instructions. 
* Also, read [this Microsoft document](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile) for Windows-specific considerations about Dockerfile instructions. 

```
# Contents of a Dockerfile: An example
FROM microsoft/windowsservercore

ENV MONGODB_VERSION 3.4.2

LABEL Description="MongoDB" Vendor="MongoDB, Inc." Version=$env:MONGODB_VERSION

ADD SOURCES /build

RUN msiexec /i c:\build\mongodb-win32-x86_64-2008plus-%MONGODB_VERSION%-signed.msi INSTALLLOCATION="C:\Program Files\MongoDB\Server\3.4.2\" /qn /l*v c:\build\mongodb-build.log & mkdir C:\data\db & mkdir C:\data\log & del  C:\build\mongodb-win32-x86_64-2008plus-%MONGODB_VERSION%-signed.msi & setx /m PATH %PATH%;"C:\Program Files\MongoDB\Server\3.4.2\bin"

VOLUME C:\\data\\db

EXPOSE 27017

CMD ["mongod.exe"]
```

Instruction/Command | Example | Description
------------|---------|------------
FROM | `FROM microsoft/windowsserverimage` | Specifies the base image. 
MAINTAINER | `MAINTAINER Ravi Sharda <ravi.sharda@gmail.com>`| Specifies the contact information for the file's author
LABEL | - | -
USER | `USER root` | By default, Docker runs all processes as root within the container. Rhis intruction sets a user. 
ENV | `ENV MONGODB_VERSION 3.4.2` | Allows you to set shell variables to be used furing the build process. Then use as %MONGODB_VERSION% inside of dos commands or $env:MONGODB_VERSION in Powershell command.
ADD | `ADD SOURCES /build` | Used to copy files (like application code, configuration files, etc.) from the local filessytem to the containers created from the image.
COPY | `COPY BlogEngine.NET_3.3_web.zip /` | Adds files from the local filesystem to the containers created from the image. 
CMD | `CMD ["mongod.exe"]` | The command that launches the process you want to have running inside of the container. A best practice is to run a single process inside of a container. 
RUN | `RUN powershell.exe C:\buildapp.ps1`| Executes a command. 
WORKDIR | `WORKDIR /data`  | Changes the working directory in the image for subsequent instructions. 
ENTRYPOINT|-| Sets the default command to be run when the container is launched.
EXPOSE|`EXPOSE 27017`| Exposes a port, so that containers can receive network requests.
And so on..|-|-
