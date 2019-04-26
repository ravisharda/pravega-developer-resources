
pravega.yaml:

```yaml
apiVersion: "pravega.pravega.io/v1alpha1"
kind: "PravegaCluster"
metadata:
  name: "pravega"
spec:
  zookeeperUri: 10.0.128.198:2181

  bookkeeper:
    image:
      repository: pravega/bookkeeper
      tag: 0.3.2
      pullPolicy: IfNotPresent

    replicas: 3

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
    segmentStoreReplicas: 3

    cacheVolumeClaimTemplate:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "default"
      resources:
        requests:
          storage: 20Gi

    image:
      repository: pravega/pravega
      tag: 0.3.2
      pullPolicy: IfNotPresent

    tier2:
      filesystem:
        persistentVolumeClaim:
          claimName: pravega-tier2

```
