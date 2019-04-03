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
* Installing IntelliJ Idea
  * https://websiteforstudents.com/install-intellij-idea-ide-on-ubuntu-16-04-17-10-18-04/
  * Using Snapd
    ```
    sudo apt install snapd
    sudo snap install intellij-idea-community --classic
    ```
## From Server on a Self-Hosted VM With Docker and Docker Compose
* Download the latest Ubuntu server edition
* Setup a VM using your favorite hypervisor: VMware Workstation or Oracle VirtualBox
* 


## From Server on Azure
* Update apt-get: ``sudo apt-get update``
* Install Ubuntu desktop: ``sudo apt-get install ubuntu-desktop``
  * See complete steps to install and connect via RDP to a Azure-hosted Ubunutu server [here](https://buildazure.com/2018/02/28/how-to-setup-an-ubuntu-linux-vm-in-azure-with-remote-desktop-rdp-access/)
* Installing Open JDK 11: 
  * See https://www.linuxuprising.com/2019/01/how-to-install-openjdk-11-in-ubuntu.html
 

## General
* Checking Ubuntu version: ``lsb_release -a``

## Troubleshooting

### Unable to install/upgrade
Say, you try to install something (say, Docker Compose: `sudo apt install docker-compose`)

If you see an error like the following:
```
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```
It means something else is being installed/upgraded. You can check that by looking at the history log:

```
less /var/log/apt/history.log
```
