---
title: Synology Keys (partial)
date: 2025-04-24 07:59:34 -400
categories: [Cheatsheets, Synology]
tags: [synology, ssh, keys] # TAG names should always be lowercase
author: mm
---

Editing needed
useradd -g scs -c "ssh user for Synology Backups" -s /bin/bash -m sshsynology
su - sshsynology -s /bin/bash
ssh-keygen -t ed25519 -b 4096
cd .ssh
cat ~/.ssh/id_ed25519.pub > ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
nano ~/.ssh/id_ed25519

Edit the 
/etc/ssh/sshd_config

Add user name to "AllowUsers" section.


