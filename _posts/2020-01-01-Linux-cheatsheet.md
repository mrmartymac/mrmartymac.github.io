---
title: Linux Cheatsheet 
date: 2023-02-20 10:39:07 -500
categories: [Cheatsheets, Linux]
tags: [linux] # TAG names should always be lowercase
---
# Linux Cheatsheet

# tar commands

## Create **tar.gz** of file or folder

```bash
tar -czvf file.tar.gz directory
```

## To extract contents of tar file

```bash
tar -xvzf my-data.tar.gz 
```

## List contents of tar file

```bash
tar -tvf my-data.tar
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
```bash
more /etc/samba/smb.conf
```

Restart SMB

```bash
sudo systemctl restart smb
```

Install smbclient

```bash
yum install samba-client
```

Check status of shares

```bash
smbclient -L
smbclient -L <LocalIP> -U <username> # Guest for example shows shares the user has permission to see.
```

## How to check if automount is running:

```bash
systemctl status autofs
```
If inactive, start with systemctl (start/restart) autofs

```bash
systemctl enable autofs
```

This will ensure that autofs starts on server start
Done.

## Configure inherited permissions with umask

Checking the current umask

```bash
umask -S
```

Adding permissions

Set a new umask value by using symbolic values with the following syntax:

```bash
umask u=#,g=#,o=#
```

Where:

u: Indicates user permissions.
g: Indicates group permissions.
o: Indicates other permissions.
#: The symbolic permission value you want to apply, as detailed in the table above.

Note: Never use space after comas when setting up a symbolic mask value.

There are also other operators you can use:
=: Creates specified file permissions and prohibits unspecified permissions.
+: Creates specified permissions, but does not change unspecified permissions.
-:Prohibits specified permissions, but does not change unspecified permissions.

Setting Up a Numeric Umask Value

Once you calculate the required umask numeric value, set it up by using:

```bash
umask [mask]
```

Where:

[mask]: The numeric value of the mask you want to apply.