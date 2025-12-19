---
title: Linux Cheatsheet 
date: 2023-02-20 10:39:07 -500
categories: [Cheatsheets, Linux]
tags: [linux] # TAG names should always be lowercase
author: mm
---
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

## To create a tar file
```bash
tar -cf <fn> file1 file2 file3
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

You can also create a file on `/etc/cron.d` with the crontab entry and it will be run as scheduled.

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

## Congifure inherited permisions with setfacl

`sudo setfacl -dm u::rwx,g::rwx,o::rwx <path>`

[Reference:](https://www.redhat.com/sysadmin/linux-access-control-lists)

Look at the current ACL
```bash
getfacl <directory>
```

Change the Default Owner of the directory
```bash
setfacl -d -m scoop:rwx <directory>
```
# Google Drive

Copy a file from a public share URL 

```bash
wget --no-check-certificate 'https://thePublicShareFromGoogleDrive' -O DestinationFileName
```

# File operations

## Using scp to copy
```bash
scp -P <ssh port> <file> <user>@<ip address>:<destination path>
```

## Using rsync to copy and update 
The `rsync` command below is configured to copy TO a remote server. The [source] is on the left and the [destination] is on the right. The destination has the destination name or IP address prepended to the remote path. The command can be used to copy TO ofr FROM a server. The username can be prepended to the destination to override authentication.

> The trailing slash is important. Using it specifies an absolute path and to place the file and directories being copied exactly there. No replication of the source path with happen.
{: .prompt-tip }
```bash
rsync -avvh /u/scoop/ <destination_ip_or_name>:/u/scoop/
```

# Find

## Find large directories

```bash
du -hs * | sort -rh | head -5
```
## Find and count large files
Removing the `| wc-l` option will return the list.  

```bash
find /u/scoop/images -type f -name *.docx -size +6M | wc -l
```
## Find files base on permissions and change the permissions
```bash
sudo find /u/scoop/images/ -type f -perm 644 -exec chmod 666 {} \;
```

## Find directories missing a file by name
This assumes the current directory and does not search deeper than one level.
```bash
for d in */; do [ ! -f "$d/<FileName>" ] && echo "${d%/}"; done
```
This version provides some more robost parameters
```bash
find . -mindepth 1 -maxdepth 1 -type d ! -exec test -f "{}/<FileName>" \; -print
```



## Move files based on time
```bash
find . -maxdepth 1 -type f -mtime +15 -exec mv {} <destPath> \;
```

## Remove files of a specific extension recursively
```bash
find . -type f -name '*.json' -delete
```

## Remove files older than some number of days
```bash
find . -mtime +60 -exec rm -f {} +
```


# Network commands

Checking for ports being listened to
```bash
dnf install mysql-server
```

# Elevation

Elevate to another user as long as I have sudo
```
sudo su - <username>
```

# Journalctl following
```bash
journalctl -fxu ScoopDaemon
```

# Journalctl for date span
```
journalctl --since "2024-06-05 23:58:00" --until "2024-06-06 00:05:00"
```

# Misc
List all installed packages and grep for one
rpm -qa | grep php

GET VERSIONS
`PHP --version`

GET LINUX VERSION
`cat /etc/redhat-release`

Remove directories with files
`rm -rf dir-name`

Linux logs
`journalctl -u ScoopDaemon | grep -i `  
`journalctl -u ScoopDaemon -n 500`

# CREATING USER
## Add scoopadmin user to the system
```bash
useradd -g scs -c "scoop admin user" -s /bin/bash -d /u/users_scs/scoopadmin -u 7000 -m scoopadmin
```

## Set password
`passwd scoopadmin`

## Create scoop Group
`groupadd scoop`

## Add user to Scoop Group
```bash
usermod -a -G scoop scoopadmin
```

## Add ScoopAdmin to sudoers
```bash
usermod -aG wheel scoopadmin
```
To enable passwordless elevation to sudo do the following
```
nano /etc/sudoers.d/<UserName>
```
Add the following to the file
```
<UserName> ALL=(ALL) NOPASSWD:ALL
```

## Enabling PHP
```bash
sudo systemctl status php-fpm
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

## Setting default linux editor
Log in to your account using SSH.
Open the `.bashrc` file in your preferred text editor.
Add the following lines to the `.bashrc` file. Replace both occurrences of program with the editor you want to set as the default editor:

```bash
export EDITOR='program'
export VISUAL='program'
```

## eLibrary permission fix for ScoopEdit
```
psql -U postgres elibrary_db
psql> grant all on all tables in schema public to scoop;
```

## Test that Windows can connect to a port
Run the following command in Powershell
```bash
test-netconnection -ComputerName scoop.newspapersystems.com -Port 7267 
```

## Changing servername or hostname
1. Check current hostname
```bash
hostnamectl
```
2. Set the new hostname (requires sudo)
```bash
sudo hostnamectl set-hostname my-new-server-name
```

3. Edit /etc/hosts (important!)
Find the line with 127.0.0.1 and change the old hostname to the new one
```bash
sudo nano /etc/hosts
```

4. Verify (may need to restart terminal/reboot)
```bash
hostnamectl
```