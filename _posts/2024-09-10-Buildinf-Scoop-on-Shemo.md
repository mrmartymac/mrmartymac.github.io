---
title: Building Scoop on Shemp
date: 2024-09-10 09:00:00 -400
categories: [Cheatsheet, Scoop]
tags: [building, shemp] # TAG names should always be lowercase
author: mm
---

## Use Screens to connect
Connect to Shemp as scsadmin.

In Terminal do the following

```bash
cd /Users/ethan/development/scoop/Installer
git remote update;
git reset --hard origin/master;
./build_scoop_mac.sh;
```

If there are errors building because of the Hunspell dictionaries, open CMakeLists.txt and change the URL for the HUNSPELL dictionaries as shown below.

```
http://cgit.freedesktop.org/libreoffice/dictionaries/plain
```
alternative
```
https://github.com/LibreOffice/dictionaries/blob/master
```