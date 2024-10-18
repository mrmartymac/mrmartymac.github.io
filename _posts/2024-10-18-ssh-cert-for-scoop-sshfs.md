---
title: Create certificate for Scoop SSHFS
date: 2024-10-18 13:12:33 -400
categories: [Cheatsheet, Scoop]
tags: [sshfs, certificate] # TAG names should always be lowercase
author: mm
---

## Enabling SSHFS on a Scoop Server

Log into the server as your user. You will need to become the `scoop` user to proceed with this process.

You will need to enter the `root` password in the next step
```bash
su;sudo -u scoop /bin/bash
```

Create an ssh key. Press enter for each of the prompts
```bash
ssh-keygen -t ed25519 -b 4096
```

You have now created the key. The contents of this key needs to be added to a file called `authorized_keys` which should exist in the `.ssh` directory scoop user's `home ` directory.

Add the key to the authorized_keys file by following these steps.
```bash
cd ~scoop/.ssh
cat id_ed25519.pub
```

Highlight and copy the resulting line(s) of text to be pasted into the `authorized_keys` file
```bash
nano authorized_keys
```

Paste the result from the above file into a new line in `authorized_keys`, save and close `authorized_keys`

Set permissions on the `authorized_keys` file
```bash
chmod 600 authorized_keys
chown scoop:scoop authorized_keys
```

Copy the file path to id_ed25519 and paste it in DatabaseManager
```bash
/home/scoop/.ssh/id_ed25519
```
![SSHFS Setup in Database Manager](/images/sshfs-setup/SSHFS-Scoop.png)
