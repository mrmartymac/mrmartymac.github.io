---
title: Configuring New Linode
date: 2024-10-02 10:42:27 -400
categories: [Documentation, Linode]
tags: [setup, server, linode, firewall] # TAG names should always be lowercase
author: mm
---
After starting the new Linode follow these steps to finish the setup.

## Configuring the Firewall

Add apache:
```bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
```
Add samba:
```bash
firewall-cmd --add-service=samba --permanent
firewall-cmd --add-service=samba-client --permanent
```
Add MOAI and CAS formula if relevant:
```bash
firewall-cmd --add-port=9006	tcp --permanent
firewall-cmd --add-port=9008	tcp --permanent
```
Add Scoop if relevant:
```bash
firewall-cmd --add-port=7267-7268	tcp --permanent
```
Reload to take effect:
```bash
firewall-cmd --reload
```
List firewall items:
```bash
firewall-cmd --list-all
```

## Collected commands for a single copy	paste

```bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --add-service=samba --permanent
firewall-cmd --add-service=samba-client --permanent
firewall-cmd --add-port=9006	tcp --permanent
firewall-cmd --add-port=9008	tcp --permanent
firewall-cmd --add-port=7267-7268	tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all
```

## Configuring fail2ban
```bash
dnf install -y epel-release
dnf install -y fail2ban fail2ban-systemd
cd 	etc	fail2ban
cp jail.conf jail.local
nano jail.local
```

### Modify jail.local:
1. Uncomment the `#ignoreip = ... `line
2. Add 10.0.0.0	24 for SCS connections
	2. Add SCS External IP addresses from Vault
	2. Add the site’s IP address as well
3. Find `backend = auto` Replace `auto` with `systemd`

Enable, Restart, Check faile2ban
```bash
systemctl restart fail2ban
systemctl enable fail2ban
systemctl status fail2ban
```
## Installing x2go
```bash
dnf --enablerepo=powertools install -y fuse-sshfs x2goserver
systemctl enable x2gocleansessions.service
systemctl restart x2gocleansessions.service
```
Make sure there are no issues with `systemctl status x2gocleansessions.service`

Install xterm
```bash
dnf install -y xterm
```
