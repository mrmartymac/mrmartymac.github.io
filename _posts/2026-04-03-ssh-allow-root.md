---
title: Allowing root from SCS
date: 2026-04-03 12:54:51 -400
categories: [Cheatsheets, Linux]
tags: [root, ssh] # TAG names should always be lowercase
author: mm
---

Adding the following to the `sshd_config` file will prohibit `root` access via ssh from anywhere other than SCS.

```bash
nano /etc/ssh/sshd_config
```
Add the following at the bottom.

```bash
# Prevent root password logins
Match User root Address *,!208.58.24.0/24
  PasswordAuthentication no

# ...unless coming from SCS
Match User root Address 208.58.24.0/24
  PasswordAuthentication yes
```
Restart `ssh`
```bash
systemctl restart sshd
```

## Change SSH port

Edit the `/etc/ssh/sshd_config` file with your preferred text editor.
```bash
nano /etc/ssh/sshd_config
```

Find the line that has `#port 22` and un-comment the line, then change 22 to the port you wish to use.  
Change:  
`#port 22`  
To:  
`port <NewPortNumber>`

Save the file. (With nano editor, press CTRL + X then Y to overwrite.)  

Restart the ssh service:
```bash
systemctl restart sshd
```

If you use iptables or the standard Linux firewall, add a rule to allow traffic to the new SSH port. (If your firewall is empty, no need.)
```bash
firewall-cmd --permanent --zone=public --add-port=<NewPortNumber>/tcp
firewall-cmd --reload
```