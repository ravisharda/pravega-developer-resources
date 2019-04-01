# Docker Compose Commands

## Quick Concepts

From [1]:
> With Compose, you use a simple text file to define an application consisting of multiple Docker containers. You then spin up your application in a single command that does everything to deploy your defined environment.

> A `docker-compose.yml` configuration file defines the Docker containers to run. The file specifies the image to run on each container, necessary environment variables and dependencies, ports, and the links between containers. 
> For details on yml file syntax, see [Compose file reference](https://docs.docker.com/compose/compose-file/).

## Commands

* Install Compose
  * `sudo apt install docker-compose`
* Checking Compose version: `docker-compose version`
* Starting the containers with Compose: From [1]: 
  > In the same directory as your `docker-compose.yml` file, run the following command (depending on your environment, you might need to run docker-compose using `sudo`.
  ```
  sudo docker-compose up -d
  
  # To verify that the containers are up
  sudo docker-compose ps
  ```
* Stopping containers: 
  * If you started Compose with docker-compose up -d, stop your services once you’ve finished with them: `sudo docker-compose stop`  
* Deleting containers:
  * You can bring everything down, removing the containers entirely, with the down command. Pass --volumes to also remove the data volume used by the container. `docker-compose down --volumes`
  
* Next Steps
  * Check out the [Compose command-line reference](https://docs.docker.com/compose/reference/) and [user guide](https://docs.docker.com/compose/) for more examples of building and deploying multi-container apps.
  * Checkout Composer File reference at https://docs.docker.com/compose/compose-file/
  * Try integrating Docker Compose with a Docker Swarm cluster. See [Using Compose with Swarm](https://docs.docker.com/compose/swarm/) for scenarios.
    

## References
1. [Azure Docker Compose Quickstart](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/docker-compose-quickstart)

