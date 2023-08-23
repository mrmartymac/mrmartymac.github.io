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
cd /u/ha/log
cat mirror_role.log 
```

Make the server Primary
```
/etc/mirror/mkprime
```
Answer NO





MURBS
Check if murbs is running.
ps -ef | grep -i murbs


systemctl status murbsd

Running full sync
murbs fullsync


FOR CRM
run mkprime
run mirror_test
restart murbsd