# Gradle Commands

## Basic
* Runnable tasks: `./gradle tasks`
* Build without executing tests: ``./gradlew build -x test``
* Install jars into local repo:
  * Install: ``./gradlew install``
  * Where to locate the new jars: Example `~/.m2/repository/io/pravega/pravega-client`
* Generate a distribution:``./gradlew distribution -x test``
* ``./gradlew clean javadocs``
* Running checkstyle for tests of standalone module: ````
* Upgrading Gradle wrapper: `gradle wrapper --gradle-version 5.6.4`
* Examples of common Gradle tasks

| Tasks | Standalone | Controller |
|:------:|:----------| :----------|
|Checkstyle|For src/main: ``./gradlew :standalone:checkstyleMain``, <br/>for src/test ``./gradlew :standalone:checkstyleTest``|``./gradlew :controller:checkstyleMain`` <br/>``./gradlew :controller:checkstyleTest``|
|Compile|``./gradlew :standalone:compileJava``|``./gradlew :controller:compileJava``|
|Tests|``./gradlew :standalone:test``|``./gradlew :controller:test ``|
|Builds|``./gradlew :standalone:build``|``./gradlew :controller:build``|
|Spotbugs|``./gradlew :standalone:spotbugsMain`` <br/>``./gradlew :standalone:spotbugsTest``|``./gradlew :controller:spotbugsMain`` <br/>``./gradlew :controller:spotbugsTest``|
* Running a specific test in a module:
```
./gradlew :module:tas; --tests "nameoftest"
Example:
# Run all tests in a test class
./gradlew :controller:test --tests "io.pravega.controller.server.rpc.auth.PravegaAuthManagerTest"

# Run a single test in a test class
./gradlew :controller:test --tests "io.pravega.controller.server.bucket.WatermarkWorkflowTest.testWatermarkClient"
./gradlew :standalone:test --tests "io.pravega.local.TlsEnabledInProcPravegaClusterTest.testWriteAndReadEventWithValidClientConfig"
```

* Local repo: 
  * Gradle: ``~/.gradle/caches/modules-2/files-2.1``
  * Maven: ``~/.m2/repository/``
  
## Running Pravega Standalone using the distribution (on Windows)
Steps:
  1. Generate the distribution (skipping tests to save time): ``gradlew distribution -x test``.
  2. Copy config files from ``pravega/config`` into ``pravega/standalone/build/install/pravega-standalone/conf``.
  3. Update any configuration as desired under the conf folder. 
  4. Start standalone by running the ``pravega/standalone/build/install/pravega-standalone/bin/pravega-standalone.bat`` file.
  
## Dependency Tree

*Dependency tree for all projects*
* Checking dependencies of a specific project:
```groovy
./gradlew <project>:dependencies
for example: ``./gradlew :controller:dependencies``
```
*``./gradle allDeps``

*Generating an HTML report (multiproject):*

* Add ``apply plugin: 'project-report'`` to allProjects
  * Add the following to allProjects
      ```groovy
      htmlDependencyReport {
        projects = project.allprojects
      }
      ```
  * Execute ``gradlew htmlDependencyReport``
  * See the reports at ``pravega\build\reports\project``
* Further reading:
  * [Interpreting the "arrow" (->) character in the output](https://stackoverflow.com/questions/27952388/what-does-arrow-mean-in-gradles-dependency-graph)

* See additional ways [here](https://stackoverflow.com/questions/21645071/using-gradle-to-find-dependency-tree/44725823). 


## Dependency checker for all projects:

*OWASP Dependency Checker:*
(Read more about it [here](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/configuration.html).)

1. Add the following highlighted lines:
    ```groovy
    buildscript {
       ....
       dependencies {
          ...
          classpath 'org.owasp:dependency-check-gradle:5.0.0' // add this line...
       }
    }
    apply plugin: 'org.owasp.dependencycheck' // ... and this one
    ```
2. Now, execute ``./gradlew dependencyCheckAggregate``
3. Look for the output at build/reports/dependency-check-report.html

## How to DOs

### Outputting stdout to a file, as well as to the console

Add the following to the task (say, to startStandalone task):
```groovy
      def fileNameTime = (new SimpleDateFormat("ssmmHH_yyyyMMdd")).format(new Date())
      doFirst {
            standardOutput = new org.apache.tools.ant.util.TeeOutputStream(
                    new FileOutputStream("$projectDir/../consoleLogs/${fileNameTime}.out"), System.out);
      }
```
## Using Artifacts from JFrog Repository

If you don't want to use released versions, you can use the [Pravega JFrog repository](https://oss.jfrog.org/artifactory/jfrog-dependencies/io/pravega/).

## Misc
```
systemProperties 'singlenode.configurationFile' : new File("$projectDir/../config/standalone-config.properties").absolutePath
```

## Maven Equivalents

| Purpose| Maven| Gradle |
|:--------:|:---|:-------|
|Find the version|`mvn --version`|`gradle --version`|
|To create JAR/WAR/EAR|`mvn package`|`gradle assemble`|
|To run unit tests|`mvn test`|`gradle test`|
|To skip unit tests|`mvn install -DskipTests`<br/>or `mvn install -Dmaven.test.skip=true`| `gradle -x test install`|
|To run JUnits and create JAR/WAR/EAR. To compile, tests and assemble.|`mvn test package`|`gradle build`|
|To clean (delete build directory)| `mvn clean` | `gradle clean`|
|To Install, i.e. to compile, build and install to local maven repository | `mvn install` | `gradle install`|
|To deploy application WAR/EAR file into server| `mvn deploy` or to run on Jetty embedded server `mvn jetty:run` | `gradle jettyRun`. See more options [here](https://www.journaldev.com/8396/gradle-vs-maven)|

## Further reading
* [Gradle vs. Maven](https://www.journaldev.com/8396/gradle-vs-maven)
