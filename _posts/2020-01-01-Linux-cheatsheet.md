---
title: Linux Cheatsheet 
date: 2023-02-20 10:39:07 -500
categories: [Cheatsheets, Linux]
tags: [linux] # TAG names should always be lowercase
---
# Linux Cheatsheet

## tar commands

### To extract contents of tar file
```bash
tar -xvzf my-data.tar.gz 
```

### List contents of tar file
```bash
tar -tvf file.tar
tar -ztvf my-data.tar.gz
tar -tvf my-data.tar.gz
tar -tvf my-data.tar.gz 'search-pattern'
```
## crontab setup
```bash
*/5 * * * * /usr/bin/sudo /usr/bin/find /u/scoop/images/ -type f -perm 644 -exec chmod 666 {} \;
```