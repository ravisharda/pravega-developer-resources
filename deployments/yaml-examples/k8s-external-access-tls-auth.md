

```yaml
apiVersion: "pravega.pravega.io/v1alpha1"
kind: "PravegaCluster"
metadata:
  name: "pravega"
spec:
  version: "0.5.0-2203.a816009-SNAPSHOT"
  zookeeperUri: 10.0.84.20:2181
  externalAccess:
    enabled: true
    type: LoadBalancer
  tls:
    static:
      controllerSecret: "controller-tls"
      segmentStoreSecret: "segmentstore-tls"
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
    serviceAccountName: pravega-components

  pravega:
    controllerReplicas: 1
    segmentStoreReplicas: 3
    controllerServiceAccountName: pravega-components
    segmentStoreServiceAccountName: pravega-components
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
        controller.auth.tlsCertFile: "/etc/secret-volume/controller01.pem"
        controller.auth.tlsTrustStore: "/etc/secret-volume/ca-cert"
        controller.auth.tlsKeyFile: "/etc/secret-volume/controller01.key.pem"
        controller.zk.secureConnection: "false"
        controller.zk.tlsTrustStoreFile: "empty"
        controller.zk.tlsTrustStorePasswordFile: "empty"
        controller.rest.tlsKeyStoreFile: "/etc/secret-volume/controller01.jks"
        controller.rest.tlsKeyStorePasswordFile: "/opt/pravega/conf/standalone.keystore.jks.passwd"
        controller.auth.enabled: "true"
        controller.auth.userPasswordFile: "/opt/pravega/conf/passwd"
        controller.auth.tokenSigningKey: "secret"
        pravegaservice.enableTls: "true"
        pravegaservice.certFile: "/etc/secret-volume/segmentStore01.pem"
        pravegaservice.keyFile: "/etc/secret-volume/segmentStore01.key.pem"
        pravegaservice.secureZK: "false"
        pravegaservice.zkTrustStore: "empty"
        pravegaservice.zkTrustStorePasswordPath: "empty"
        autoScale.tlsEnabled: "true"
        autoScale.tlsCertFile: "/etc/secret-volume/segmentStore01.pem"
        autoScale.validateHostName: "false"
        autoScale.authEnabled: "true"
        autoScale.tokenSigningKey: "secret"
        bookkeeper.tlsEnabled: "false"
        bookkeeper.tlsTrustStorePath: "empty"
```
