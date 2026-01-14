---
title: Docker Update 
date: 2025-12-18 12:15:15 -400
categories: [Cheatsheets, Linux]
tags: [docker, novnc] # TAG names should always be lowercase
author: mm
---


## Check if docker is installed.  

```
docker ps
```
  
* If you get "docker" not found, nothing is currently installed and there is nothing to worry about.  
* If you get a list of docker sessions or even an empty list, docker is installed.  
  
Check if you have a trouble version.  
## Check for sessions.  

```
docker ps
```
  
CONTAINER ID   IMAGE     COMMAND       CREATED        STATUS        PORTS     NAMES  
e3b6e0aaa50c   novnc     "/bin/bash"   24 hours ago   Up 24 hours             CAS_1  
ffaaef8b303e   novnc     "/bin/bash"   26 hours ago   Up 26 hours             CAS_2  
fc6a83446907   novnc     "/bin/bash"   7 days ago     Up 7 days               CAS_21  
3e25fd3e20eb   novnc     "/bin/bash"   7 days ago     Up 7 days               CAS_22  
8a22e136fec2   novnc     "/bin/bash"   7 days ago     Up 7 days               CAS_23  
8c9ec1c4d753   novnc     "/bin/bash"   7 days ago     Up 7 days               CAS_24  
c7f5b61b46fd   novnc     "/bin/bash"   8 days ago     Up 8 days               CAS_14  
710a1398f79a   novnc     "/bin/bash"   8 days ago     Up 8 days               CAS_7  
b793b7047558   novnc     "/bin/bash"   8 days ago     Up 8 days               CAS_12  
## Join a session.  

```
docker exec -it CAS_12 /bin/bash
```
  
## Check for the installed docker version.  

```
cat /etc/redhat-release
```
  
AlmaLinux release 9.7 (Moss Jungle Cat)  
## Leave the session.  

```
exit
```
  
  
If you do not see CentOS7 above, then you don't need to do anything else. If you do see CentOS7, then you will need to upgrade the OS version for Docker.  
  
## Go to the novnc directory.  

```
cd /u/moai/novnc
```
  
## Get the installer is for almalinux9.  

```
ncftpget -u ftpaccess -p knuckle ftp.newspapersystems.com . ethan/install_docker.sh
```
  
## Good to go but you'll need to make sure you can take SCS down for about 30 minutes to an hour.  
## Kill all active docker sessions.  

```
docker stop $(docker ps -q)
```
  
## Now install a new version.  

```
./install_docker.sh
```
  
  
## Note, if you do not have an active docker session to check the OS, you can start your own.  

```
cd /u/moai/novnc
```
  
## Test that the new version is installed
```bash
docker exec -it $(docker run -dit --network=host novnc) /bin/bash
```
```bash
cat /etc/redhat-release
```

We are looking for `AlmaLinux release 9.x`