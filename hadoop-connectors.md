# Hadoop Connectors Recipes 

## Building the project with a specifed Pravega Version (Instead of with the Pravega Sub-module)

1. Clone a branch of Pravega from the master or your fork that you want to use.
2. Publish Maven artifacts of Pravega to local repo by executing `./gradlew clean install`. 
3. Note the version of the Pravega artifacts by inspecting `~/.m2/repository/io/pravega`. Say, it was `0.6.0-2259.b80b233-SNAPSHOT`.  
4. Checkout Hadoop Connectors repo: `git clone --recursive https://github.com/pravega/hadoop-connectors.git`. 
5. Update gradle.properties of the Hadoop Connectors project: look for pravega artifact version variable and update the artifact version to the one that you just published locally. Ensure that the project doesn't use the Pravega sub-module by setting the appropriate flag.
   
   ```
   pravegaVersion=0.6.0-2259.b80b233-SNAPSHOT
   usePravegaVersionSubModule=false
   ```
6. Build the Haddop connectors project again: `./gradlew clean build`.

## Using Hadoop Connectors

### Verifying whether Auth works

1. Enable Auth by setting `enableAuth` flag as true in `io.pravega.connectors.hadoop.utils.SetupUtils` class.
2. Ensure that the code is recompiled: `./gradlew clean build -x test`. 
3. Run a subset of tests by interactively running tests in `io.pravega.connectors.hadoop.PravegaConnectorLocalJobITCase` class. 

Alternatively, run all tests in step 2 itself, by executing `./gradlew clean build` instead.

### Running tests with it pointing to a separate Pravega Deployment

```
./gradlew clean test -Dpravega.uri=tls://localhost:9090
```
