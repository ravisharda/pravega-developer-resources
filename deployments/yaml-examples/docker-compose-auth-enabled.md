
Here are the steps: 

1. Prepare a YAML manifest file, like the one shown below. We are enabling `auth` (short for authentication and authorization) in this. 

   `docker-compose.yml`: 

   ```yaml

  version: "3"
  services:
    zookeeper:
      image: zookeeper:3.5.4-beta
      ports:
        - "2181:2181"

    hdfs:
      image: pravega/hdfs:2.7.7
      ports:
        - "2222:2222"
        - "8020:8020"
        - "50090:50090"
        - "50010:50010"
        - "50020:50020"
        - "50075:50075"
        - "50070:50070"
      environment:
        SSH_PORT: 2222
        HDFS_HOST: ${HOST_IP}

    bookie1:
      image: pravega/bookkeeper:0.5.1
      ports:
        - "3181:3181"
      restart: always
      environment:
        ZK_URL: zookeeper:2181
        bookiePort: 3181
      links:
        - zookeeper

    bookie2:
      image: pravega/bookkeeper:0.5.1
      ports:
        - "3182:3182"
      restart: always
      environment:
        ZK_URL: zookeeper:2181
        bookiePort: 3182
      links:
        - zookeeper

    bookie3:
      image: pravega/bookkeeper:0.5.1
      ports:
        - "3183:3183"
      restart: always
      environment:
        ZK_URL: zookeeper:2181
        bookiePort: 3183
      links:
        - zookeeper

    controller:
      image: pravega/pravega:0.5.1
      ports:
        - "9090:9090"
        - "10080:10080"
      command: controller
      environment:
        WAIT_FOR: zookeeper:2181
        ZK_URL: zookeeper:2181
        REST_SERVER_PORT: 10080
        JAVA_OPTS: |
          -Dcontroller.service.port=9090
          -Dcontroller.auth.tlsEnabled=false
          -Dcontroller.auth.tlsCertFile="/pki/server-cert.crt"
          -Dcontroller.auth.tlsTrustStore="/pki/ca-cert.crt"
          -Dcontroller.auth.tlsKeyFile="/pki/server-key.key"
          -Dcontroller.zk.secureConnection=false
          -Dcontroller.zk.tlsTrustStoreFile="/pki/zk.truststore.jks"
          -Dcontroller.zk.tlsTrustStorePasswordFile="/pki/zk.truststore.jks.password"
          -Dcontroller.rest.tlsKeyStoreFile="/pki/server.keystore.jks"
          -Dcontroller.rest.tlsKeyStorePasswordFile="/pki/server.keystore.jks.passwd"
          -Dcontroller.auth.enabled=true
          -Dcontroller.auth.userPasswordFile="/opt/pravega/conf/passwd"
          -Dconfig.controller.metricenableCSVReporter=false
          -Dcontroller.auth.tokenSigningKey=secret
          -Xmx512m
          -XX:OnError="kill -9 p%"
          -XX:+ExitOnOutOfMemoryError
          -XX:+CrashOnOutOfMemoryError
          -XX:+HeapDumpOnOutOfMemoryError
        SERVICE_HOST_IP: segmentstore
      links:
        - zookeeper

    segmentstore:
      image: pravega/pravega:0.5.1
      ports:
        - "12345:12345"
      command: segmentstore
      depends_on: 
        - hdfs
      environment:
        WAIT_FOR: bookie1:3181,bookie2:3182,bookie3:3183,hdfs:8020
        TIER2_STORAGE: "HDFS"
        HDFS_REPLICATION: 1
        HDFS_URL: ${HOST_IP}:8020
        ZK_URL: zookeeper:2181
        CONTROLLER_URL: tcp://${HOST_IP}:9090
        JAVA_OPTS: |
          -Dpravegaservice.enableTls=false
          -Dpravegaservice.enableTlsReload=false
          -Dpravegaservice.certFile="/pki/server-cert.crt"
          -Dpravegaservice.keyFile="/pki/server-key.key"
          -Dpravegaservice.secureZK=false
          -Dpravegaservice.zkTrustStore="/pki/zk.truststore.jks"
          -Dpravegaservice.zkTrustStorePasswordPath="/pki/zk.truststore.jks.password"
          -DautoScale.tlsEnabled=false
          -DautoScale.tlsCertFile="/pki/server-cert.crt"
          -DautoScale.validateHostName=false
          -DautoScale.authEnabled=true
          -DautoScale.tokenSigningKey=secret
          -Dbookkeeper.tlsEnabled=false
          -Dbookkeeper.tlsTrustStorePath="/pki/bk.truststore.jks"
          -Dmetrics.enableCSVReporter=false
          -Dpravegaservice.publishedIPAddress=${HOST_IP}
          -Dbookkeeper.bkEnsembleSize=2
          -Dbookkeeper.bkAckQuorumSize=2
          -Dbookkeeper.bkWriteQuorumSize=2
          -Dpravega.client.auth.token="YWRtaW46MTExMV9hYWFh"
          -Dpravega.client.auth.method="Basic"
          -Xmx900m
          -XX:OnError="kill -9 p%"
          -XX:+ExitOnOutOfMemoryError
          -XX:+CrashOnOutOfMemoryError
          -XX:+HeapDumpOnOutOfMemoryError
      links:
        - zookeeper
        - hdfs
        - bookie1
        - bookie2
        - bookie3

   ```

2. Create an environment variable pointing to the current host IP address: `$ export HOST_IP=<host_IP>`

3. Change directory to the one containing `docker-compose.yml` file: `$ cd /path/to/docker-compose-manifest-file`

4. Deploy the application specified in the `docker-compose.yml` file: `$ docker-compose up -d`

5. Verify that the application is up and running. The output of the following command should print the response body shown below. 

   ```
   $ curl -v -u admin:1111_aaaa http://$HOST_IP:10080/v1/scopes 
   ```

   Expected Output: 
   
   ```
   {"scopes":[{"scopeName":"_system"}]}
   ```

For other Docker Compose commands, see [this](https://github.com/ravisharda/pravega-developer-resources/blob/master/deployments/docker-compose-commands.md). 
