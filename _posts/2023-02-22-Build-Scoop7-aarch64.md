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
/u/SCOOP/scoop_development
```
Generate the key for SSH to be added to github.com - **NOTE** Use the default and do not rename the file. Take note of the ID.numeric name.  
```
cd /home/<username>/.ssh
ssh-keygen -t ed25519 -C "marty@newspapersystems.com"
```
Provide a filename for the first request, suggested `id_rsa`.  Press enter twice for the passphrase.  

Run the less command on the key `id_rsa.pub` (if you used the suggested name) to copy and paste into github.com.  The key will start with `ssh-ed25519`

```
less ~/.ssh/<keyname>.pub
```
Copy the string so that you can paste it in to gethub.com.  
[https://github.com/settings/keys](https://github.com/settings/keys)  

```
cd /u/SCOOP/scoop_development
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