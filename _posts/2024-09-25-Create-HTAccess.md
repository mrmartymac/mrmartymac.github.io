---
title: How to create .htaccess protection to secure web directories:
date: 2024-09-25 11:04:26 -400
categories: [Cheatsheet, Apache]
tags: [security, htaccess] # TAG names should always be lowercase
author: mm
---
Create the .htpasswd file along with a new user and password
```bash
sudo htpasswd -cb /etc/httpd/.htpasswd <user> <pass>
```
-c is the option to create a new file
-b says to use the password provided
If you exclude this, you will be prompted for the password at the command line
Replace <user> with the user you want to use
Replace <pass> with the password you want to use  

Allow apache to read the .htpasswd file
```bash
sudo chmod 664 /etc/httpd/.htpasswd
```  

Create the .htaccess file in the directory you want
For example, for PageTrack, we'll create it at /u/srv/pagetrack/.htaccess
It should look like the following:
```
AuthType Basic
AuthName "Restricted Location"
AuthUserFile /etc/httpd/.htpasswd
Require valid-user 
```
Modify the relevant file or directory and change `AllowOverride None` to `AllowOverride All` to allow .htaccess to work
    a. For PageTrack, you'll want to modify the top `<Directory>` block in `/etc/httpd/conf.d/pagetrack.conf`