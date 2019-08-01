## Setting up Cert Manager

The quick-start guide will get you running within minutes.
https://github.com/jetstack/cert-manager/blob/master/docs/tutorials/acme/quick-start/index.rst#step-5---deploy-cert-manager. You could follow step 5 to install the cert-manager chart, and then create an issuer and certificate using the snippets on this page:

https://docs.cert-manager.io/en/latest/tasks/issuers/setup-ca.html This would cause a Secret to be generated (containing the server certificate pair), to be referenced by the segmentStoreSecret field of the PravegaCluster.


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
