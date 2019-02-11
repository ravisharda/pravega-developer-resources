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

Assumption: the certificate is in ``cert.pem`` file. 

|S.No.|Command|Description|
|:----:|:-------|:--------|
|1.|``openssl x509 -outform der -in cert.pem -out cert.der``| Convert the PEM certificate into a DER file. |
|2.|``keytool -import -v -trustcacerts -alias caroot -file cert.der -keystore standalone.truststore.jks``| Create the standalone.truststore.jks truststore file and add the server certificate to the truststore. Enter password 1111_aaaa at the "Enter Keystore Password" prompt.|
|3.| ``keytool -list -v -keystore standalone.keystore.jks``| List the truststore's contents to verify everything is in order.|
   
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
* Check a particular keystore entry using an alias: ``keytool -list -v -keystore bookie.truststore.jks -alias caroothjhhj
* List trusted CA Certs: 
  ```
  keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts
  keytool -list -v -keystore /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/cacerts
  
  # List details about a particular CA cert in the Trusted CA Certs list by alias:
  keytool -list -v -keystore /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/cacerts -alias local-CA
  ```

*Further Reading:*
* https://stackoverflow.com/questions/13732826/convert-pem-to-crt-and-key

## Creating PKI Infrastructure for a Cluster
1. Generate the key and the certificate for each component in the cluster - in this case just the standalone server. 
   
   Note that the server certificate is used by server and verified by client for server identity. You can export the key from the keystore file later and sign it later with CA.

   ```
   keytool -keystore standalone.server.keystore.jks\
      -genkey -keyalg RSA -keysize 2048 -keypass 1111_aaaa\
      -alias localhost -validity 3650\
      -dname "CN=localhost, OU=standalone, O=Pravega, L=Seattle, S=Washington, C=US"\
      -storepass 1111_aaaa
      
   # Add "-ext SAN=DNS:<hostname>" if needed.
      
   # Optionally, list the keystore's contents to verify everything is in order. 
   # The outout will show 1 entry in the keystore with - Alias name: localhost, Entry type: PrivateKeyEntry.
   keytool -list -v -keystore standalone.server.keystore.jks -storepass 1111_aaaa
   ```
2. Create a Certificate Authority (CA). 
   ```
   # a) Generate a CA, which in turn is public-private key pair and certificate. The CA will be used 
   # to sign other certificates. 
   openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650 \
               -subj "/C=US/ST=Washington/L=Seattle/O=Pravega/OU=CA/CN=CA" \
               -passin pass:1111_aaaa -passout pass:1111_aaaa
   
   # b) Add the generated CA to a new truststore for use by the clients.
   keytool -keystore standalone.client.truststore.jks -noprompt -alias CARoot -import -file ca-cert \
        -storepass 1111_aaaa
   
   # Optionally, list the truststore's contents to verify everything is in order. Expected 1 entry with
   # Alias name=caroot, entry type: trustedCertEntry
   keytool -list -v -keystore standalone.client.truststore.jks -storepass 1111_aaaa
  
   # c) Add the generated CA certificate to a new certificate for use by the server components. 
   keytool -keystore standalone.server.truststore.jks -noprompt -alias CARoot -import -file ca-cert \
       -storepass 1111_aaaa    
       
   # Optionally, list the truststore's contents to verify everything is in order.  Expected 1 entry with
   # Alias name=caroot, entry type: trustedCertEntry
   keytool -list -v -keystore standalone.server.truststore.jks -storepass 1111_aaaa
   ```
3. Now, sign the server's certificates using the generated CA. 
   
   ```
   # a) Export the certificate from the keystore:
   keytool -keystore standalone.server.keystore.jks -alias localhost -certreq -file signing-request.csr \
        -storepass 1111_aaaa
        
   # Optionally, to view the contents of the generated file, execute: 
   openssl req -in signing-request.csr -noout -text
   
   # b) Sign the exported certificate by the CA.
   openssl x509 -req -CA ca-cert -CAkey ca-key -in signing-request.csr -out server-cert-signed.pem \
        -days 3650 -CAcreateserial -passin pass:1111_aaaa
        
   #c) Import the CA certificate and the server's signed certificate into the server's keystore:  
   
   keytool -keystore standalone.server.keystore.jks -alias CARoot -noprompt \
              -import -file ca-cert -storepass 1111_aaaa
              
   keytool -keystore standalone.server.keystore.jks -alias localhost -noprompt \
              -import -file server-cert-signed.pem -storepass 1111_aaaa
   
   # Now, check the server keystore to see everything is in order. Expect two entries at this point: 
   #     1) Alias name = caroot, entry type = trustedCertEntry
   #     2) Alias name = localhost, entry type = PrivateKeyEntry
   keytool -list -v -storepass 1111_aaaa -keystore standalone.server.keystore.jks
   ```
 
 4. Export the server's key from the server keystore.
 
   ```
   # a) Convert the .jks file into a pkcs12 file:
   keytool -importkeystore -srckeystore standalone.server.keystore.jks  \
                           -destkeystore standalone.server.keystore.p12 \
                           -srcstoretype jks -deststoretype pkcs12 \
                           -srcstorepass 1111_aaaa -deststorepass 1111_aaaa
                           
   # b) Export the Private Key into a key.pem PEM file. Here we are creating a pem file with no password   
   # If you use "openssl pkcs12 -in standalone.server.keystore.p12 -out key.pem -passin pass:1111_aaaa 
   # -passout pass:1111_aaaa", the key.pem file will have a pem password as well apart from key encryption 
   # password. We use -nodes flag to avoid that. 
   openssl pkcs12 -in standalone.server.keystore.p12 -out key.pem -passin pass:1111_aaaa -nodes
   
   # Check the key using: 
   openssl pkcs8 -inform PEM -in key.pem -topk8 -passin pass:1111_aaaa
   ```
     
 5. Export the server's certificate from the server keystore. Actually, you don't really need to this, as you already have it
    in the form of ``server-cert-signed.pem``. Just rename it to cert.pem. 
    ```
    mv server-cert-signed.pem cert.pem
    ```
                          
6. Do a final verification. Run the following commands and compare the modulus. If the modulus of the all three command 
   matches, you should be good to go.  

   ```
   # For the key:
   openssl rsa -noout -modulus -in key.pem
   
   # For the certificate:
   openssl x509 -noout -modulus -in cert.pem
   
   # For the certificate signing request
   openssl req -noout -modulus -in signing-request.csr
   ```


*Further Reading:*
* https://docs.confluent.io/current/tutorials/security_tutorial.html#generating-

s-certs
* https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.1.0/configuring-wire-encryption/content/create_and_set_up_an_internal_ca_openssl.html

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

## Further Reading
* [Most Common OpenSSL Commands by SSL Shopper](https://www.sslshopper.com/article-most-common-openssl-commands.html)
* [Most Common Keytool Commands by SSL Shopper](https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html)
* Creating certs and keystore/truststores
  * [Glassfish - Generate a Certificate Using Keytool](https://docs.oracle.com/cd/E19798-01/821-1751/ghlgv/index.html)
  * [Confluent Platform - Encryption and Authentication with SSL](https://docs.confluent.io/current/kafka/authentication_ssl.html)
  * Creating certificates and keys for a omponent: [HortonWorks Data Platform - Create and Set Up an Internal CA OpenSSL](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.1.0/configuring-wire-encryption/content/create_and_set_up_an_internal_ca_openssl.html)
  * https://stackoverflow.com/questions/47434877/how-to-generate-keystore-and-truststore
