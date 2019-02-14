# Ubuntu VM Setup

## From Desktop
* Install Git: ``sudo apt install git``
* Install Gradle: 
  
  ```
  wget https://services.gradle.org/distributions/gradle-3.4.1-bin.zip
  sudo mkdir /opt/gradle
  sudo unzip -d /opt/gradle gradle-3.4.1-bin.zip
  export PATH=$PATH:/opt/gradle/gradle-3.4.1/bin
  ```
* Install Open JDK 8

  ```
  sudo add-apt-repository ppa:openjdk-r/ppa
  sudo apt-get update
  sudo apt-get install openjdk-8-jdk
  ```

## From Server on Azure
* Update apt-get: ``sudo apt-get update``
* Install Ubuntu desktop: ``sudo apt-get install ubuntu-desktop``
