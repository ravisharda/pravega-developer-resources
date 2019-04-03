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

## Miscellaneous
* Want to use a self-hosted/private registry? 
  ```
  docker login devops-repo.isus.emc.com:8116
  
  # Then continue to use other commands
  ```
# Deploying Pravega Distrubuted Cluster Using Docker Compose

Note: Unlike pravega-standalone, a Docker Compose cluster will use a real standalone HDFS, Zookeeper, BookKeeper, and will run the Segment Store and the Controller in separate processes.

## Prerequisites
* Linux VM with Git, Docker and Docker Compose Installed
* Ensure you don't have to use `sudo` everywhere by first executing `sudo -s`

## Steps to Deploy
1. Clone Pravega repo: `git clone https://github.com/pravega/pravega.git`.
2. `cd ./pravega/docker/compose`
3. In this directory, you'll see a `docker-compose.yml` configuration file for the deployment. Now, run the following command to setup the dployment: 
   
   ```
   export HOST_IP=192.168.224.129 && docker-compose up -d
   
   Or, if you have already exported HOST_IP:
   docker-compose up -d
   
   This works better:
   docker-compose up -d --force-recreate
   
   # Note: 
   #  - The -d flag runs it in the background
   #  - Replace the IP address with the host's IP Address.
   ```
4. To verify that the containers are up: `docker-compose ps`
5. To check the logs: `docker-compose logs` (You'll see all the containers' logs here)

## Steps to Undeploy

Prerequisites: 
* Ensure you are in ./pravega/docker/compose. 
* Ensure you don't have to use `sudo` everywhere by first executing `sudo -s`

1. First, stop the running containers: `docker-compose stop`
2. Second remove the stopped containers: `docker-compose rm -f`

   Note: If you don’t specify -f in the above command, it will prompt you for Y/N before removing it.


You can combine the two steps like this: 
```
# Since we have “&&”, which will execute the 2nd command only after the 1st command is successful.

docker-compose stop && docker-compose rm -f -v

Or even, just (-s stop if required before removing)
docker-compose rm -f -s -v

```

Note: 
* Removing volumes: While removing a stopped containers, it doesn’t remove all the volumes that are attached to the containers. In a typical situation, you don’t want to remove the attached volumes during your regular stop/start/rm process. But, if you decide to remove the attached volumes, you can do that during rm by using -v option as shown below. [3]

```
docker-compose rm -v

To Check volumes:
docker volume ls -qf dangling=true

To remove dangling volumes:
docker volume rm $(docker volume ls -qf dangling=true)
```
   
## Troubleshooting

### Starting a shell inside the container

```
# Run a shell inside the Controller container
docker exec -it compose_controller_1 sh

# Run a shell inside the Segmentstore container
docker exec -it compose_segmentstore_1 sh
```

### Checking logs 
```
# Check logs for the entire deployment
docker-compose logs

# Check logs of a particular container
docker logs compose_controller_1
-- See last 100 lines
docker logs compose_controller_1 --tail 100
-- See last 100 lines and keep following the logs (tail -100f ...)
docker logs compose_controller_1 --tail 100 -f
```

### Error upon stopping 

You might see an error like this, after you execute docker-compose stop. 
```
root@server2:~/pravega/docker/compose# docker-compose stop
Stopping compose_segmentstore_1 ... error
Stopping compose_bookie1_1      ... error
Stopping compose_controller_1   ... error
Stopping compose_bookie2_1      ... error
Stopping compose_hdfs_1         ... error
Stopping compose_zookeeper_1    ... error

ERROR: for compose_segmentstore_1  cannot stop container: b465df6cd8e014dc22a01ca14713da6e3e3d0f90fa9be97f943cc0ad7ebaadc5: Cannot kill container b465df6cd8e014dc22a01ca14713da6e3e3d0f90fa9be97f943cc0ad7ebaadc5: unknown error after kill: runc did not terminate sucessfully: container_linux.go:388: signaling init process caused "permission denied"
: unknown

ERROR: for compose_controller_1  cannot stop container: 4330dc574264187b36ef19376600731bc2f2898a7b391cd20913b594000a8ab8: Cannot kill container 4330dc574264187b36ef19376600731bc2f2898a7b391cd20913b594000a8ab8: unknown error after kill: runc did not terminate sucessfully: container_linux.go:388: signaling init process caused "permission denied"
: unknown

```

The solution is (as per [this](https://forums.docker.com/t/can-not-stop-docker-container-permission-denied-error/41142/3) discussion: 
```
Check status: sudo aa-status

Shutdown and prevent it from restarting: sudo systemctl disable apparmor.service --now
Unload AppArmor profiles: sudo service apparmor teardown
Check status: sudo aa-status

You should now be able to stop/kill containers.
````

## References & Further Reading
1. https://github.com/pravega/pravega/tree/master/docker/compose
2. https://hub.docker.com/u/pravega/
3. https://www.thegeekstuff.com/2016/04/docker-compose-up-stop-rm/comment-page-1/  

## References
1. [Azure Docker Compose Quickstart](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/docker-compose-quickstart)

