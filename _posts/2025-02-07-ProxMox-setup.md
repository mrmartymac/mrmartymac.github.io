---
title: Setting up ProXom VM
date: 2025-02-07 11:35:08 -400
categories: [ProxMox, Setup]
tags: [vm, proxmox, virtual, network] # TAG names should always be lowercase
author: mm
---

## Adding DHCP address
After createing the VM, to get it to have an internal IP address at SCS do the following.

Log into the GUI desktop session
1. Click on the Settings in the top right corner  
2. Click on the Settings icon  
![System Settings](/images/proxmox/ProxMox-DHCP1.png)
3. Choose Network  
4. Turn on Wired  
5. Click the Gear icon  
![Activate wired connection](/images/proxmox/ProxMox-DHCP2.png)
6. Enable **Connect Automatcially**
7. Configure IPv4 for DHCP  
![Configure DHCP](/images/proxmox/ProxMox-DHCP3.png)

## Set the hostname
Use the following commands to set the hostname.

### Hostnamectl
To see the current hostname run
```bash
hostnamectl
```

To set the name to the name you want, use the command below. 
```bash
hostnamectl set-hostname your-desired-hostname
```

There is also a `--pretty` option
```bash
hostnamectl set-hostname "examplecloud production server" --pretty
```

### Edit /etc/hosts
Edit `/etc/hosts` to add the name(s) and address(es) of the instances.
```bash
nano /etc/hosts
```

### Reloading the new network settings.
```bash
systemctl restart systemd-hostnamed
```
