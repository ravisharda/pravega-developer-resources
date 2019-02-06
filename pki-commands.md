# PKI Commands

## Displaying the contents of certificates and keystores/truststores

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
