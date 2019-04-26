You will find sample app definition/specification files for distributed mode deployment of Pravega using Docker Compose and Kubernetes. 

## Docker Compose

### Sample Docker Compose Files

#### Sample 1: TLS + Auth in Controllers and Segment Stores only

NOTE:
* Zookeeper and Bookkeeper run without TLS, in this case.
* Certificates and keys are created upfront (through a process) and shared from the host to the containers via a directory attached as volume in the container

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
    image: pravega/bookkeeper:0.5.0-2203.a816009-SNAPSHOT
    ports:
      - "3181:3181"
    restart: always
    environment:
      ZK_URL: zookeeper:2181
      bookiePort: 3181
    links:
      - zookeeper

  bookie2:
    image: pravega/bookkeeper:0.5.0-2203.a816009-SNAPSHOT
    ports:
      - "3182:3182"
    restart: always
    environment:
      ZK_URL: zookeeper:2181
      bookiePort: 3182
    links:
      - zookeeper

  bookie3:
    image: pravega/bookkeeper:0.5.0-2203.a816009-SNAPSHOT
    ports:
      - "3183:3183"
    restart: always
    environment:
      ZK_URL: zookeeper:2181
      bookiePort: 3183
    links:
      - zookeeper

  controller:
    image: pravega/pravega:0.5.0-2203.a816009-SNAPSHOT
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
        -Dcontroller.auth.tlsEnabled=true
        -Dcontroller.auth.tlsCertFile="/pki/controller01.pem"
        -Dcontroller.auth.tlsTrustStore="/pki/ca-cert"
        -Dcontroller.auth.tlsKeyFile="/pki/controller01.key.pem"
        -Dcontroller.zk.secureConnection=false
        -Dcontroller.zk.tlsTrustStoreFile="/pki/zk.truststore.jks"
        -Dcontroller.zk.tlsTrustStorePasswordFile="/pki/zk.truststore.jks.password"
        -Dcontroller.rest.tlsKeyStoreFile="/pki/controller01.jks"
        -Dcontroller.rest.tlsKeyStorePasswordFile="/pki/password"
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
    volumes:
      - /home/rsharda/pki:/pki
    links:
      - zookeeper

  segmentstore:
    image: pravega/pravega:0.5.0-2203.a816009-SNAPSHOT
    ports:
      - "12345:12345"
    command: segmentstore
    environment:
      WAIT_FOR: bookie1:3181,bookie2:3182,bookie3:3183,hdfs:8020
      TIER2_STORAGE: "HDFS"
      HDFS_REPLICATION: 1
      HDFS_URL: ${HOST_IP}:8020
      ZK_URL: zookeeper:2181
      CONTROLLER_URL: tcp://${HOST_IP}:9090
      JAVA_OPTS: |
        -Dpravegaservice.enableTls=true
        -Dpravegaservice.certFile="/pki/segmentstore01.pem"
        -Dpravegaservice.keyFile="/pki/segmentstore01.key.pem"
        -Dpravegaservice.secureZK=false
        -Dpravegaservice.zkTrustStore="/pki/zk.truststore.jks"
        -Dpravegaservice.zkTrustStorePasswordPath="/pki/zk.truststore.jks.password"
        -DautoScale.tlsEnabled=true
        -DautoScale.tlsCertFile="/pki/segmentstore01.pem"
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
        -Xmx900m
        -XX:OnError="kill -9 p%"
        -XX:+ExitOnOutOfMemoryError
        -XX:+CrashOnOutOfMemoryError
        -XX:+HeapDumpOnOutOfMemoryError
    volumes:
      # Attach the local directory containing PKI material to the container
      - /home/rsharda/pki:/pki
    links:
      - zookeeper
      - hdfs
      - bookie1
      - bookie2
      - bookie3
```

## Kubernetes

### Sample Files

File for deploying Zookeper cluster using Zookeeper Operator (zk.yaml)

```
apiVersion: "zookeeper.pravega.io/v1beta1"
kind: "ZookeeperCluster"
metadata:
  name: "pravega-zookeeper"
spec:
  size: 3
```

File for deploying Pravega Cluster using Pravega Operation (pravega.yaml)

```yaml
apiVersion: "pravega.pravega.io/v1alpha1"
kind: "PravegaCluster"
metadata:
  name: "pravega"
spec:
  zookeeperUri: 10.0.249.248:2181

  bookkeeper:
    replicas: 3
    image:
      repository: shardar/bookkeeper
      tag: 0.5.0-2203.a816009-SNAPSHOT
      pullPolicy: IfNotPresent
    storage:
      ledgerVolumeClaimTemplate:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "default"
        resources:
          requests:
            storage: 10Gi

      journalVolumeClaimTemplate:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "default"
        resources:
          requests:
            storage: 10Gi
    autoRecovery: true

  pravega:
    controllerReplicas: 1
    segmentStoreReplicas: 1
    cacheVolumeClaimTemplate:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "default"
      resources:
        requests:
          storage: 20Gi
    image:
      repository: shardar/pravega
      tag: 0.5.0-2203.a816009-SNAPSHOT
      pullPolicy: IfNotPresent
    tier2:
      filesystem:
        persistentVolumeClaim:
          claimName: pravega-tier2
    options:
        controller.auth.tlsEnabled: "true"
        controller.auth.tlsCertFile: "/opt/pravega/conf/cert.pem"
        controller.auth.tlsTrustStore: "/opt/pravega/conf/cert.pem"
        controller.auth.tlsKeyFile: "/opt/pravega/conf/key.pem"
        controller.zk.secureConnection: "false"
        controller.zk.tlsTrustStoreFile: ""
        controller.zk.tlsTrustStorePasswordFile: ""
        controller.rest.tlsKeyStoreFile: "/opt/pravega/conf/standalone.keystore.jks"
        controller.rest.tlsKeyStorePasswordFile: "/opt/pravega/conf/standalone.keystore.jks.passwd"
        controller.auth.enabled: "true"
        controller.auth.userPasswordFile: "/opt/pravega/conf/passwd"
        controller.auth.tokenSigningKey: "secret"
        pravegaservice.enableTls: "true"
        pravegaservice.certFile: "/opt/pravega/conf/cert.pem"
        pravegaservice.keyFile: "/opt/pravega/conf/key.pem"
        pravegaservice.secureZK: "false"
        pravegaservice.zkTrustStore: ""
        pravegaservice.zkTrustStorePasswordPath: ""
        autoScale.tlsEnabled: "true"
        autoScale.tlsCertFile: "/opt/pravega/conf/cert.pem"
        autoScale.validateHostName: "false"
        autoScale.authEnabled: "true"
        autoScale.tokenSigningKey: "secret"
        bookkeeper.tlsEnabled: "false"
        bookkeeper.tlsTrustStorePath: ""
```


