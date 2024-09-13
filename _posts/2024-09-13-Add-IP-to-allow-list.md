---
title: Add/Remove IP to allowed list
date: 2024-09-13 11:20:36 -400
categories: [Cheatsheet, Networking]
tags: [allow, allowlist, whitelist, firewall, firewalld] # TAG names should always be lowercase
author: mm
---
The following steps show how to add/remove IP addresses to the Allowed List for connections.

Change directory to
```bash
cd /etc/firewalld/services
```

Command to list the IPs currently configured in `firewalld`
```bash
firewall-cmd --list-all
```

Command to add an IP address to a given service. In this case the example is using the service named `ssh-custom`. This could vary from site to site and there may be a variety of rules for different services like SSH, FTP, etc.
```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="208.58.24.113/29" service name="ssh-custom" accept'
```

After updating the rules, the service has to be reloaded for the changes to take effect.
```bash
firewall-cmd --reload
```

Various other commands noted below.  
```bash
firewall-cmd --permanent --zone=public --add-service=ftp (port 21)
```
```bash
firewall-cmd --permanent --add-port=10090-10100/tcp
```
```bash
firewall-cmd --reload
```