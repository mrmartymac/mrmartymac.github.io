---
title: Setting up VSFTP
date: 2025-05-29 07:21:50 -400
categories: [Cheatsheet, FTP]
tags: [ftp, vstp] # TAG names should always be lowercase
author: mm
---
## Adding a user
Create the user account
```bash
useradd -c "User Description" -d <HomeDirectoryPath> -m -s /bin/ftponly <userName>
```
Set the password for the user
```bash
passwd <userName>
```
Add the user to the user_list and save.
```bash
nano /etc//etc/vsftpd/user_list
```


## Raw history
```bash
systemctl status ncftpd
firewall-cmd --add-service=ftp --permanent --zone=public
ln -s /usr/lib/systemd/system/vsftpd.service /etc/systemd/system/multi-user.target.wants/vsftpd.service 
ln -s /etc/systemd/system/multi-user.target.wants/vsftpd.service /usr/lib/systemd/system/vsftpd.service
systemctl restart vsftpd
more ftp-custom.xml 
more /etc/vsftpd/vsftpd.conf
nano /etc/vsftpd/vsftpd.conf
systemctl restart vsftpd
nano /etc/vsftpd/vsftpd.conf
systemctl status vsftpd
nano /etc/vsftpd/vsftpd.conf
systemctl restart vsftpd
passwd csn_ftp
cd /etc/vsftpd/
nano vsftpd.conf
systemctl restart vsftpd
cd /u/ftp/
cd /u/ftp/CSN/
clear ; ftp
cd ftp
nano /etc/vsftpd/vsftpd.conf
nano /etc/vsftpd/user_list 
history | grep csn_ftp
useradd -c "SCS FTP" -d /u/ftp/SCS -m -s /bin/ftponly scs_ftp
mkdir /u/ftp/SCS
chown scs_ftp:scs /u/ftp/SCS
passwd scs_ftp
cd /u/ftp/
deluser scs_ftp
rmuser scs_ftp
userdel -r scs_ftp
nano /etc/vsftpd/user_list 
```