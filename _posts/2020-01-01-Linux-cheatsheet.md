---
title: Linux Cheatsheet 
date: 2023-02-20 10:39:07 -500
categories: [Cheatsheets, Linux]
tags: [linux] # TAG names should always be lowercase
---
# Linux Cheatsheet

[Welcome 3](#welcome)

[Launcher 6](#launcher)

[i. Installing 6](#installing)

[ii. Signing in 6](#signing-in)

[iii. Application Launcher 8](#application-launcher)

[iv. User menu 9](#user-menu)

[ScoopEdit 10](#scoopedit)

[i. Main window 10](#main-window)

[1. Basket list 11](#basket-list)

[2. Article list 12](#article-list)

[3. Status bar 14](#status-bar)

[4. Toolbar 15](#toolbar)

[5. Article info 16](#article-info)

[6. Attachment list 16](#attachment-list)

[7. Article preview 18](#article-preview)

[8. Other features 19](#other-features)

[ii. Locating articles 19](#locating-articles)

[1. Find in articles 20](#find-in-articles)

[2. Search in all baskets 21](#search-in-all-baskets)

[iii. Creating articles 23](#creating-articles)

[iv. Editing articles 23](#editing-articles)

# tar commands

## To extract contents of tar file
```bash
tar -xvzf my-data.tar.gz 
```

## List contents of tar file
```bash
tar -tvf file.tar
tar -ztvf my-data.tar.gz
tar -tvf my-data.tar.gz
tar -tvf my-data.tar.gz 'search-pattern'
```
# crontab setup
This is the example to schedule a command to conditionally change the permissions on a set of files every five minutes.
```bash
*/5 * * * * /usr/bin/sudo /usr/bin/find /u/scoop/images/ -type f -perm 644 -exec chmod 666 {} \;
```