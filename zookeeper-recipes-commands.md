# Zookeeper Recipes and Commands

## Building Zookeeper

```bash
$ git clone -b branch-3.5.5 https://github.com/apache/zookeeper
$ cd zookeeper
$ mvn package -DskipTests
$ mvn install -DskipTests
```

## Using Zookeeper

### With SSL/TLS enabled

1. Rename the config/zoo_sample.cfg to config/zoo.cfg.
2. Create a server_envs.sh file with the server environment variables containing:

   ```
   export SERVER_JVMFLAGS="
     -Dzookeeper.serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
     -Dzookeeper.ssl.keyStore.location=/path/to/pravega/config/server.keystore.jks 
     -Dzookeeper.ssl.keyStore.password=1111_aaaa 
     -Dzookeeper.ssl.trustStore.location=/path/to/pravega/config/client.truststore.jks
     -Dzookeeper.ssl.trustStore.password=1111_aaaa" 
  ```
3. Execute `$ source server_envs.sh`.
