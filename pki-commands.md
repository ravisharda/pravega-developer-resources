# PKI Commands

## Displaying the contents of certificates, keys and keystores/truststores

* Displaying the contents of certificates using OpenSSL
  * A .pem file: ``openssl x509 -in /path/to/cert.pem -text``
  * A .crt file: ``openssl x509 -in /path/to/ca-certificates.crt -text``
  * A DER file: ``openssl x509 -in MYCERT.der -inform der -text``

* Displaying the contents of certificates, using Keytool
  * ``keytool -printcert -v -file mydomain.crt``
  * ``keytool -printcert -v -file cert.pem``
  
* Displaying the contents of keystores/truststores using Java Keytool command
  ```
  keytool -list -v -keystore <file-name.jks>, then enter keystore password on the prompt
   
  Example:
  keytool -list -v -keystore bookie.truststore.jks
  ```
   
* Displaying the contents of a file containing private key using OpenSSL
  ```
  Example:
  openssl pkcs8 -inform PEM -in key.pem -topk8 (then enter password on the prompt)
  ```
   
* Displaying the common name (CN) of a certificate: ``openssl x509 -noout -subject -in cert.pem``
  ``` 
  Sample output:
  subject=C = AU, ST = Some-State, O = Internet Widgits Pty Ltd, CN = localhost
   ```
 
## Importing the provided CA certificate into local JVM's truststore

1. Change directory to config directory: ``cd /path/to/pravega/config`` 
2. Convert the provided 'cert.pem' file to DER format: ``openssl x509 -in cert.pem -inform pem -out cert.der -outform der``
3. Import the certificate into the local JVM's trust store: 
   ```sudo keytool -importcert -alias local-CA -keystore /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/cacerts -file cert.der``` (default password is "changeit")
   
## Adding an existing server certificate into a truststore file

AssumptionL the certificate is in ``cert.pem`` file. 

|S.No.|Command|Description|
|:----:|:-------|:--------|
|1.|``openssl x509 -outform der -in cert.pem -out cert.der``| Convert the PEM certificate into a DER file. |
|2.|``keytool -import -v -trustcacerts -alias caroot -file cert.der -keystore standalone.truststore.jks``| Create the standalone.truststore.jks truststore file and add the server certificate to the truststore.|
   
## Adding an existing server certificate and key into a keystore file

Assumption: the key is in a password-protected ``key.pem`` file and the certificate is in ``cert.pem`` file. 

|S.No.|Command|Description|
|:--:|:-------|:---------|
|1.| ``openssl rsa -in key.pem -out keyout.pem`` | Remove password from ``key.pem`` file. |
|2.| ```openssl pkcs12 -export -out key.pfx -inkey keyout.pem -in cert.pem -name localhost``` (Enter an appropriate export password, when prompted.) |  Create a PKCS12 file containing the key. |
|3.| ``keytool -importkeystore -deststorepass 1111_aaaa -destkeystore standalone.keystore..jks -srckeystore key.pfx -srcstoretype PKCS12`` | Convert the pfx file to jks file. |
|4.| ``keytool -list -v -keystore standalone.keystore.jks`` | Verify the jks file.|
|5.|-| Import the cert into the keystore.|
|6.|``keytool -list -v -keystore standalone.keystore.jks`` | Verify the jks file.|

(Note: PKCS12 files tend to have extensions .pfx or .p12)

## Certificate Related

``openssl x509 -noout -subject -in cert.pem``

## Keys Related

* Remove the pass phrase on an RSA privat key: ``openssl rsa -in key.pem -out keyout.pem``, then inspect the file.
* Convert a private key from PEM to DER format: ``openssl rsa -in key.pem -outform DER -out keyout.der``
* Print out the components of a private key to standard output: ``openssl rsa -in key.pem -text -noout``
* Just output the public part of a private key: ``openssl rsa -in key.pem -pubout -out pubkey.pem``
* Output the public part of a private key in RSAPublicKey format: ``openssl rsa -in key.pem -RSAPublicKey_out -out pubkey.pem``
* Extract all the certs, including the CA Chain: ``openssl crl2pkcs7 -nocrl -certfile key.pem | openssl pkcs7 -print_certs -out key.cert``
* Extracting the key: ``openssl rsa -in foo.pem -out foo.key``

## Keystore/Trustore-Specific

* Change an alias of an entry in the JKS store:

  ```
  keytool -changealias -alias "caroot" -destalias "<new_alias>" -keystore <keystore_name>
  ```

* Delete an entry with a given alias: ``keytool -delete -alias caroot -keystore standalone.keystore.jks``
* Checking which certificates are in Java Keystore: ``keytool -list -v -keystore bookie.truststore.jks``
* Check a particular keystore entry using an alias: ``keytool -list -v -keystore bookie.truststore.jks -alias caroot``
* List trusted CA Certs: 
  ```
  keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts
  keytool -list -v -keystore /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/cacerts
  
  # List details about a particular CA cert in the Trusted CA Certs list by alias:
  keytool -list -v -keystore /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/cacerts -alias local-CA
  ```
* 

*Further Reading:*
* https://stackoverflow.com/questions/13732826/convert-pem-to-crt-and-key

## Hitting a TLS-Protected Service using cURL or OpenSSL

```
curl -v -k https://localhost:9091/v1/scopes (-k == --insecure)


# Use TLS 1.1
curl -v -k -tlsv1.1 -cacert cert.pem https://localhost:9091/v1/scopes

# Check an SSL connection. All the certificates (including Intermediates) should be displayed
openssl s_client -connect www.paypal.com:443

openssl s_client -showcerts -connect localhost:9091

openssl s_client -help
```

## General OpenSSL Commands

* Checking OpenSSL Version: ``openssl version -a``
* 


## Further Reading
* [Most Common OpenSSL Commands by SSL Shopper](https://www.sslshopper.com/article-most-common-openssl-commands.html)
* [Most Common Keytool Commands by SSL Shopper](https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html)
* Creating certificates and keys for a omponent:
  * [HortonWorks Data Platform - Create and Set Up an Internal CA OpenSSL](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.1.0/configuring-wire-encryption/content/create_and_set_up_an_internal_ca_openssl.html)

