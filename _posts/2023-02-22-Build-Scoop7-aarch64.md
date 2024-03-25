---
title: How to build Scoop for aarch64 on Alma9
date: 2023-02-22 15:40:23 -500
categories: [Documentation, Builds]
tags: [server, build, daemon] # TAG names should always be lowercase
author: mm
---
Instructions on building Scoop on the Parallels hosted AlmaLinux server which is an AlmaLinux 9 aarch64 version to support my M1 processor.

## Creating initial clone 
Create the directory  
```
/u/SCOOP/scoop_development/scoop
```
Generate the key for SSH to be added to github.com - **NOTE** Use the default and do not rename the file. Take note of the ID.numeric name.  
```
ssh-keygen -t ed25519 -C "marty@newspapersystems.com"
```
Run the less command on the key to copy and paste into github.com  
```
less ~/.ssh/<keyname>
```
Copy the string so that you can paste it in to gethub.com.  
[https://github.com/settings/keys](https://github.com/settings/keys)  

```
git clone git@github.com:newspapersystems/scoop.git
```

## Updating yum repository
```bash
cd /u/SCOOP/scoop_development/scoop
git remote update
git reset --hard origin/master
./Installer/build_scoop.sh
```

## Running actual update
```bash
cd Installer/build/dist
```
Become su
```bash
yum update ScoopDaemon X.X.X (Be sure to get the server RPM and not the client.)
```