---
title: Linux Cheatsheet 
date: 2023-02-20 10:39:07 -500
categories: [Cheatsheet, Linux]
tags: [linux] # TAG names should always be lowercase
author: mm
---
# Linux Cheatsheet

# Configure your profile to use Nano as the default editor

First, insatall Nano

`yum install nano`

Edit your .bash_profile

`nano ~/.bash_profile`

Add the user preference

`export VISUAL="nano"`
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

To edit crontab 
`crontab -e`

> Runtime parameters. `minute hour day_of_month month day_of_week command_to_run`
{: .prompt-info }


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

# Configure inherited permissions with umask

Checking the current umask

```bash
umask -S
```

Changing umask permanently requires changes to be made the the user's **/home/\<username>/.bash_profile**

How to Calculate Files and Directories Permissions Based on UMASK Value

As you might be aware, base permission of a file is 0666 and base permission of a directory is 0777. Hence final permission of files and directories will get calculated from this base permission values. If umask is set to 0022 in the system, then creating a file and directories will have below permission.


> For Files : 0666 - 0022 = 0644
{: .prompt-info }
> For Directories: 0777 - 0022 = 0755
{: .prompt-info }

If umask is set 0032, then creating file and directories will have below permission.

>For Files: 0666 - 0032 = 0634
{: .prompt-info }
>For Directories: 0777 - 0032 = 0745
{: .prompt-info }

## Open your .bash_profile
`nano /home/\<userName>/.bash-profile`

### For `rw-rw-rw-` on **files** and `rwxrwxrwx` on **directories**
```bash
umask 0000
```

### For `rw-rw-r--` on **files** and `rwxrwxr--` on **directories**
```bash
umask 0002
```

> Be certain to log off nad back in before testing the changes to your .bash_profile.  The change to your umask will not take effect until the user logs out and back in.
{: .prompt-warning }

# Congifure inherited permisions with setfacl

`sudo setfacl -dm u::rwx,g::rwx,o::rwx <path>`
# Google Drive

Copy a file from a public share URL 

```bash
wget --no-check-certificate 'https://thePublicShareFromGoogleDrive' -O DestinationFileName
```

# File operations

## Find large directories

```bash
du -a | sort -n -r | head -n 5
```

## Move files based on time
```bash
find . -maxdepth 1 -type f -mtime +15 -exec mv {} <destPath> \;
```
