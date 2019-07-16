# REST API Operations Usage Examples

  * [Usage Instructions](#usage-instructions)
  * [Listing all scopes (Auth Enabled, TLS Disabled)](#listing-all-scopes--auth-enabled--tls-disabled-)
    + [Sample Request](#sample-request)
    + [Sample response](#sample-response)
  * [Creating a new Scope](#creating-a-new-scope)
    + [Sample Request](#sample-request-1)
    + [Sample Response](#sample-response)
  * [Add a Stream to a Scope](#add-a-stream-to-a-scope)
    + [Sample Request](#sample-request-2)
    + [Sample Response](#sample-response-1)
  * [Listing Streams of a Scope](#listing-streams-of-a-scope)
    + [Sample Request](#sample-request-3)
    + [Sample Response](#sample-response-2)

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

## Listing all scopes (Auth Enabled, TLS Disabled)

### Sample Request

```
GET {{protocol}}://{{hostname}}:{{port}}/v1/scopes

Headers:
  Authorization: Basic YWRtaW46MTExMV9hYWFh

Ex: GET http://localhost:9091/v1/scopes
```

### Sample response

```
Status: 200 OK

Response Body: 
{"scopes":[{"scopeName":"_system"}]}
```

## Creating a new Scope

### Sample Request

```
POST {{protocol}}://{{hostname}}:{{port}}/v1/scopes
Ex: POST http://localhost:9091/v1/scopes

Headers:
  Authorization: Basic YWRtaW46MTExMV9hYWFh
  Content-Type: application/json
  Accept: application/json

Request Body:
{
    "scopeName": "org.example.myscope"
}
```

### Sample Response

```
Status: 201 Created

Response Body:
{
  "scopeName": "org.example.myscope"
}
```

## Add a Stream to a Scope

### Sample Request

```
POST {{protocol}}://{{hostname}}:{{port}}/v1/scopes/{{myscope}}/streams
Ex: POST http://localhost:9091/v1/scopes/org.example.myscope/streams

Headers:
  Authorization: Basic YWRtaW46MTExMV9hYWFh
  Content-Type: application/json
  Accept: application/json

Request Body:
{
  "streamName" : "mystream",
  "scalingPolicy" : {
    "type" : "FIXED_NUM_SEGMENTS",
    "targetRate" : 0,
    "scaleFactor" : 0,
    "minSegments" : 1
  }
}
```

### Sample Response

```
Status: 201 Created

Response Body:
{
    "scopeName": "org.example.myscope",
    "streamName": "mystream",
    "scalingPolicy": {
        "type": "FIXED_NUM_SEGMENTS",
        "minSegments": 1
    }
}

```

## Listing Streams of a Scope

### Sample Request

```
GET {{protocol}}://{{hostname}}:{{port}}/v1/scopes/{{myscope}}/streams/
Ex: GET http://localhost:9091/v1/scopes/org.example.myscope/streams

Headers:
  Authorization: Basic YWRtaW46MTExMV9hYWFh

```

### Sample Response

```
Status: 200 OK

Response Body:

{
    "streams": [
        {
            "scopeName": "org.example.myscope",
            "streamName": "mystream2",
            "scalingPolicy": {
                "type": "FIXED_NUM_SEGMENTS",
                "minSegments": 1
            }
        },
        {
            "scopeName": "org.example.myscope",
            "streamName": "mystream",
            "scalingPolicy": {
                "type": "FIXED_NUM_SEGMENTS",
                "minSegments": 1
            }
        }
    ]

```
