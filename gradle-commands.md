# Gradle Commands

## Basic
* Build without executing tests: ``./gradlew build -x test``
* Install jars into local repo:``./gradlew install``
* Generate a distribution:``./gradlew distribution -x test``
* ``./gradlew clean javadocs``
* Examples of common Gradle tasks

| Tasks | Standalone | Controller |
|:------:|:----------| :----------|
|Checkstyle| ./gradlew :standalone:checkstyleMain|./gradlew :controller:checkstyleMain|
|Compile| ./gradlew :standalone:compileJava| ./gradlew :controller:compileJava |
|Tests|./gradlew :standalone:test|./gradlew :controller:test  |
|Builds|./gradlew :standalone:build|./gradlew :controller:build |
* Running a specific test in a module:
```
./gradlew :module:tas; --tests "nameoftest"
Example:
./gradlew :controller:test --tests "io.pravega.controller.server.rpc.auth.PravegaAuthManagerTest"
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
```
./gradlew <project>:dependencies`` 
for example: ``./gradlew :controller:dependencies``
```
*``./gradle allDeps``

*Generating an HTML report (multiproject):*

* Add ``apply plugin: 'project-report'`` to allProjects
  * Add the following to allProjects
      ```
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
    ```
    buildscript {
       ....
       dependencies {
          ...
          classpath 'org.owasp:dependency-check-gradle:4.0.2' // add this line...
       }
    }
    apply plugin: 'org.owasp.dependencycheck' // ... and this one
    ```
2. Now, execute ``./gradlew dependencyCheckAggregate``.
3. Look for the output at build/reports/dependency-check-report.html
