# Docker Compose Commands

## Quick Concepts

From [1]:
> With Compose, you use a simple text file to define an application consisting of multiple Docker containers. You then spin up your application in a single command that does everything to deploy your defined environment.

> A `docker-compose.yml` configuration file defines the Docker containers to run. The file specifies the image to run on each container, necessary environment variables and dependencies, ports, and the links between containers. 
> For details on yml file syntax, see [Compose file reference](https://docs.docker.com/compose/compose-file/).

## Commands

* Install Compose
  * `sudo apt install docker-compose`
* Starting the containers with Compose: From [1]: 
  > In the same directory as your `docker-compose.yml` file, run the following command (depending on your environment, you might need to run docker-compose using `sudo`.
  ```
  sudo docker-compose up -d
  
  # To verify that the containers are up
  sudo docker-compose ps
  ```
    

## References
1. [Azure Docker Compose Quickstart](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/docker-compose-quickstart)

