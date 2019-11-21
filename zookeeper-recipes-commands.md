# Zookeeper Recipes and Commands

## Building Zookeeper

```bash
# Replace branch name with the branch you want to build.
$ git clone -b branch-3.5.5 https://github.com/apache/zookeeper
$ cd zookeeper
$ mvn package -DskipTests
$ mvn install -DskipTests
```

## Using Zookeeper

### Enable SSL/TLS 

1. Download the binary: `curl https://archive.apache.org/dist/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz --output apache-zookeeper-3.5.5-bin.tar.gz`

   For other versions, check https://archive.apache.org/dist/zookeeper/. 
   
2. `tar -xvzf apache-zookeeper-3.5.5-bin.tar.gz`
3. `cd apache-zookeeper-3.5.5-bin/`
4. Rename the config/zoo_sample.cfg to config/zoo.cfg.
5. Create a server_envs.sh file with the server environment variables containing:

   ```
   export SERVER_JVMFLAGS="
     -Dzookeeper.serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
     -Dzookeeper.ssl.keyStore.location=/path/to/pravega/config/server.keystore.jks 
     -Dzookeeper.ssl.keyStore.password=1111_aaaa 
     -Dzookeeper.ssl.trustStore.location=/path/to/pravega/config/client.truststore.jks
     -Dzookeeper.ssl.trustStore.password=1111_aaaa" 
   ```
6. Execute `$ source server_envs.sh`.
7. Edit the conf/zoo.cfg by adding secureClientPort=2281
8. Create a client_envs.sh file containing: 
   
   ```
   export CLIENT_JVMFLAGS="
      -Dzookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty 
      -Dzookeeper.client.secure=true 
      -Dzookeeper.ssl.keyStore.location=/path/to/pravega/config/server.keystore.jks 
      -Dzookeeper.ssl.keyStore.password=1111_aaaa 
      -Dzookeeper.ssl.trustStore.location=/path/to/pravega/config/client.truststore.jks 
      -Dzookeeper.ssl.trustStore.password=1111_aaaa"
   ```
9. Execute `$ source client_envs.sh` on the client side.

### Starting/Stopping the Server and the CLI client

```bash
$ bin/zkServer.sh start
$ bin/zkCli.sh -server localhost:2281

# Verify client is able to talk to the server CLI is able to communicate with the server.
$ 
```

### Links
* https://issues.apache.org/jira/browse/ZOOKEEPER-2125
* https://cwiki.apache.org/confluence/display/ZOOKEEPER/ZooKeeper+and+SASL
* https://cwiki.apache.org/confluence/display/ZOOKEEPER/ZooKeeper+SSL+User+Guide


