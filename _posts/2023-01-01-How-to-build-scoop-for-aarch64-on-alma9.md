---
title: How to build Scoop for aarch64 on Alma9
date: 2023-03-17 14:05:25 -400
categories: [Cheatsheet, Build]
tags: [build] # TAG names should always be lowercase
author: mm
---
# How to build Scoop for aarch64 on Alma9

Instructions on building Scoop on the Parallels hosted AlmaLinux server which is an AlmaLinux 9 aarch64 version to support my M1 processor.

## Updating yum repository

1.	`cd /u/SCOOP/scoop_development/scoop`
2.	`git remote update`
3.	`git reset --hard origin/master`
4.	`./Installer/build_scoop.sh`

## Running actual update

1.	`cd Installer/build/dist`
2.	Become su
3.	`yum update ScoopDaemon X.X.X` (Be sure to get the server RPM and not the client.)