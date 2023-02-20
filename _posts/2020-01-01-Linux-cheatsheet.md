---
title: Linux Cheatsheet 
date: 2023-02-20 10:39:07 -500
categories: [Cheatsheets, Linux]
tags: [linux] # TAG names should always be lowercase
---
# Linux Cheatsheet

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

# smb

## Check SMB Status

Ask the server for the SMB shares it has
```terminal
more /etc/samba/smb.conf
```

Restart SMB

```console
sudo systemctl restart smb
```

Install smbclient

```yaml
yum install samba-client
```

Check status of shares

```ksrc
smbclient -L
smbclient -L <LocalIP> -U <username> # Guest for example shows shares the user has permission to see.
```

## How to check if automount is running:

```
systemctl status autofs
```
If inactive, start with systemctl (start/restart) autofs

```
systemctl enable autofs
```

This will ensure that autofs starts on server start
Done.
