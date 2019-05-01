# Performing basic verification of TLS and Auth

**Pre-Requisites:**
* You have a cluster running with TLS and auth enabled. 
* Note the HOST IP of the controller service you want to connect to. 

1. The following command should result in a response containing a system scope `_system`.
    
    ```bash
    # Containerized deployments use port 10080 by default. 9091 is the default port in other cases. 
    $ curl -v -k -u admin:1111_aaaa https://$HOST_IP:10080/v1/scopes
    ```
      
 2. The following command with invalid credentials should result in a `401 Unauthorized` response.
    
    ```bash
    $ curl -v -k -u whatadmin:whoever https://$HOST_IP:10080/v1/scopes
    ```
    
3. Run a reader/writer client program. The one I'm using can be found here: 

   https://github.com/ravisharda/pravega-examples/blob/6b2ff2a5f0373d67dee1133eb829b40d62ab69f4/src/main/java/org/example/pravega/client/basicreadwrite/ReaderWriterExamples.java#L28
