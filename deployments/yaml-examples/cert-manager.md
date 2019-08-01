

## Issuer yaml

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: ca-issuer
  namespace: default
spec:
  ca:
    secretName: ca-key-pair
```

## Segment Store Cert Request YAML example

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: segmentstore-cert
  namespace: default
spec:
  secretName: segmentstore-tls
  keyEncoding: pkcs8
  duration: 2h
  renewBefore: 1h
  issuerRef:
    name: ca-issuer
    kind: Issuer
  commonName: segmentstore.pravega.io
  organization:
  - Example CA
  dnsNames:
  - segmentstore.pravega.io
  - segmentstore2.pravega.io
```
