# REST API Operations Usage Examples

## Usage Instructions

1. Replace parameters contained in brackets (`{{ }}`) with your values applicable to your own environment. For example, for listing all scopes, here's a sample endpoint for a TLS-enabled Controller endpoint.  
   * Generic: `{{protocol}}://{{hostname}}:{{port}}/v1/scopes`
   * Specific: `https://localhost:9091/v1/scopes`

2. Depending on the tool you use and the AuthHandler implementation, use the appropriate command. For example, if you are using `curl` and the default PasswordAuthHandler for authentication which uses basic authentication, the commands for listing scopes may be as follows:

  * TLS and auth enabled, with the signing CA certificate not trusted on the client host: 
    
    ```
    $ curl -v -k -u admin:1111_aaaa https://localhost:9091/v1/scopes
    ```
    Note that this was a GET request, so there's no request body. Also, see curl documentation to understand the flags. 
    
  * TLS disabled, auth enabled: 
    
    ```
    $ curl -v -u admin:1111_aaaa http://localhost:9091/v1/scopes
    ```
    
  * Both TLS and auth disabled:
  
    ```
    $ curl -v http://localhost:9091/v1/scopes
    ```
  
  
    
   



## Listing all scopes

### Request

```
GET {{protocol}}://{{hostname}}:{{port}}/v1/scopes
```

### Response


