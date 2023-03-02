---
title: How to build Scoop for aarch64 on Alma9
date: 2023-02-22 15:40:23 -500
categories: [Documentation, Builds]
tags: [server, build, daemon] # TAG names should always be lowercase
---
Instructions on building Scoop on the Parallels hosted AlmaLinux server which is an AlmaLinux 9 aarch64 version to support my M1 processor.

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