

```yaml
apiVersion: "pravega.pravega.io/v1alpha1"
kind: "PravegaCluster"
metadata:
  name: "pravega"
spec:
  version: "0.5.0-2203.a816009-SNAPSHOT"
  zookeeperUri: pravega-zookeeper-client:2181
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
        controller.auth.tlsCertFile: "/etc/secret-volume/controllerTlsCertFile"
        controller.auth.tlsTrustStore: "/etc/secret-volume/controllerCacert"
        controller.auth.tlsKeyFile: "/etc/secret-volume/controllerTlsKeyFile"
        controller.zk.secureConnection: "false"
        controller.zk.tlsTrustStoreFile: "empty"
        controller.zk.tlsTrustStorePasswordFile: "empty"
        controller.rest.tlsKeyStoreFile: "/etc/secret-volume/controllerTlsKeyStoreFile"
        controller.rest.tlsKeyStorePasswordFile: "/etc/secret-volume/passwordfile"
        controller.auth.enabled: "true"
        controller.auth.userPasswordFile: "/opt/pravega/conf/passwd"
        controller.auth.tokenSigningKey: "secret"
        pravegaservice.enableTls: "true"
        pravegaservice.certFile: "/etc/secret-volume/segmentstoreTlsCertFile"
        pravegaservice.keyFile: "/etc/secret-volume/segmentstoreTlsKeyFile"
        pravegaservice.secureZK: "false"
        pravegaservice.zkTrustStore: "empty"
        pravegaservice.zkTrustStorePasswordPath: "empty"
        autoScale.tlsEnabled: "true"
        autoScale.tlsCertFile: "/etc/secret-volume/segmentstoreTlsCertFile"
        bookkeeper.tlsEnabled: "false"
        bookkeeper.tlsTrustStorePath: "empty"
        autoScale.validateHostName: "false"
        autoScale.authEnabled: "true"
        autoScale.tokenSigningKey: "secret"
        pravega.client.auth.token: "YWRtaW46MTExMV9hYWFh"
        pravega.client.auth.method: "Basic"
```
