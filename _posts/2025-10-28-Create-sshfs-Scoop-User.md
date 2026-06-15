---
title: Creating SSH-FS Keys for Scoop user
date: 2025-10-28 08:37:33 -400
categories: [Cheatsheets, SMB]
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

Be sure to `cat` the public key into a file called `authorized_keys`
```bash
cat id_ed25519.pub > authorized_keys
```

Ensure the permissions of the `authorized_keys` and `id_ed25519` are `600`
```bash
chmod 600 authorized_keys id_ed25519
```

Configure the **SSHFS Settings** as shown below entering the IP address and SSH port for the server.
 ![SSHFS Settings](/images/scoop-sshfs-settings/Scoop-SSHFS-Settings.png)