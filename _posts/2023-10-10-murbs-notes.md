---
title: Notes about Mirroring and MURBS
date: 2023-08-10 13:55:51 -400
categories: [Cheatsheet, Mirroring]
tags: [mirroring, murbs] # TAG names should always be lowercase
author: mm
---

# Narrative
Singular or multiple DR sites is possible.

## Paths
`/etc/mirror` contains the mirroring configurations
Primary and Secondary

Determine which server is the primary

Run as root or sudo  
```
sudo su -
```

Test the current status
```
/etc/mirror/mirror.test
```

There is a better way to test,Use this to test in crontab.  It is run as the mirror user. It removes the requirement to be root to run the test.
/etc/mirror/mirror_test



## How to determine which is primary and which is secondary
Look in mirror.conf to see if this method is used.

## Checking the current or previous role
```
cat /u/ha/log/mirror_role.log 
```

Make the server Primary
```
/etc/mirror/mkprime
```
Answer NO





MURBS
Check if murbs is running.
```
ps -ef | grep -i murbs
```

systemctl status murbsd

Running full sync
```
murbs fullsync
```

FOR CRM
run mkprime
run mirror_test
restart murbsd

## October 11, 2023
### Resetting murbs at CRM  
Become root
```bash
sudo su -
```
Stop murbs
```bash
systemctl stop murbsd
```

### On DR server  
Remove any residual `gz` files 
```bash
rm -f /etc/murbs/work/postgres/in/*.gz
```

### On Primary  
Remove the `error` file, start `murbsd` and run a `fullsync`
```bash
rm /etc/murbs/error
systemctl start murbsd
murbs fullsync
```

## Checking Logs
To check the nightly log look at copy_fs.log
```bash
more /u/ha/log/copy_fs.log
```

## NOTE about journaltctl error about 10 files  
It seems this is a red herring and is not a problem.  Murbs is detecting the presence of multiple past postgresql log dumps and using this as the trigger to clean them up