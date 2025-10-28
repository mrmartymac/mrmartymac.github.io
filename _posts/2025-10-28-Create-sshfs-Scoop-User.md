---
title: Creating SSH-FS Keys for Scoop user
date: 2025-10-28 08:3732:33 -400
categories: [Cheatsheet, SMB]
tags: [smb, ssh, sshfs, keygen. keys] # TAG names should always be lowercase
author: mm
---

## Steps to create the user on the server

We will follow these steps to create the ssh keys for the already present `scoop` user.


Become the scoop user.
```bash
sudo su - scoop -s /bin/bash
```

Create an SSH key. Pres enter each time you are prompted taking the defaults.
```bash
ssh-keygen -t ed25519 -b 4096
```

Configure the **SSHFS Settings** as shown below entering the IP address and SSH port for the server.
 ![SSHFS Settings](/images/scoop-sshfs-settings/Scoop-SSHFS-Settings.png)