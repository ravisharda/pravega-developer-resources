# PKI Commands

## Displaying the contents of certificates, keys and keystores/truststores

* Displaying the contents of certificates using OpenSSL
  * A .pem file: ``openssl x509 -in /path/to/cert.pem -text``
  * A .crt file: ``openssl x509 -in /path/to/ca-certificates.crt -text``
  
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
   
## Adding an existing server certificate and key into a keystore file

Assumption: the key is in a password-protected ``key.pem`` file and the certificate is in ``cert.pem`` fil. 

|S.No.|Command|Description|
|:--:|:-------|:---------|
|1.| ``openssl rsa -in key.pem -out keyout.pem`` | Remove password from ``key.pem`` file. |
|2.| ``openssl pkcs12 -export -out key.pfx -inkey keyout.pem -in cert.pem -name localhost`` (Enter an appropriate export password, when prompted.) |  Create a PKCS12 file containing the key. |
|3.| ``keytool -importkeystore -deststorepass 1111_aaaa -destkeystore standalone.keystore..jks -srckeystore key.pfx -srcstoretype PKCS12`` | Convert the pfx file to jks file. |

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


*Further Reading:*
* https://stackoverflow.com/questions/13732826/convert-pem-to-crt-and-key

## Further Reading
* [Most Common OpenSSL Commands - SSL Shopper](https://www.sslshopper.com/article-most-common-openssl-commands.html)
