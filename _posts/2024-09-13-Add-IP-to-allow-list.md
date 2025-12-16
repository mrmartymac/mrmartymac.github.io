---
title: Add/Remove IP to allowed list
date: 2024-09-13 11:20:36 -400
categories: [Cheatsheets, Networking]
tags: [allow, allowlist, whitelist, firewall, firewalld] # TAG names should always be lowercase
author: mm
---
The following steps show how to add/remove IP addresses to the Allowed List for connections.

## List the IPs currently configured in the firewall  
Take note of the **service names** in use.
```bash
firewall-cmd --list-all
```

## Adding IP Addresses

IP addresses are added to a given service. In this case the example is using the service named `ssh-custom`. This could vary from site to site and there may be a variety of rules for different services like SSH, FTP, etc.  

Replace `<IP-Address-to-add>` with the IP address you are adding. if the IP address is a range you can define it with the CIDR notation. For example the CIDR notation for a range from 1.1.1.0 - 1.1.1.3 would be 1.1.1.0/30.

> Here is a helpful [CIDR Calculator](https://mxtoolbox.com/subnetcalculator.aspx)  
{: .prompt-tip }

The value afer `service name=` may vary. Ensure the service name is correct. You can find the service names when you run the `firewall-cmd --list-all` command.  
```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="<IP-Address-to-add>" service name="ssh-custom" accept'
```

## Reloading the firewall rules after changes  
You change will not take effect until you reload the firewall rules.  
```bash
firewall-cmd --reload
```

## Removing an entry
```
firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="<IP-Address-To-Remove>" service name="ssh-custom" accept'
```
## Misc commands
Adding a service name
```bash
firewall-cmd --permanent --zone=public --add-service=ftp (port 21)
```
Adding port ranges
```bash
firewall-cmd --permanent --add-port=10090-10100/tcp
```
Configuration file location
```bash
cd /etc/firewalld/services
```