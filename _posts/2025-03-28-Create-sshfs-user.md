---
title: Creating SSH-FS user
date: 2025-03-28 10:37:33 -400
categories: [Cheatsheet, SMB]
tags: [smb, ssh, sshfs, keygen. keys] # TAG names should always be lowercase
author: mm
---

## Steps to create the user on the server

On the server you need to create a user for the sshfs connection. We are using `sshfs-samba` as the user name. Please create the user with the following commands and parameters.

Become root.

```bash
useradd -c "sshfs user" -s /bin/true -m sshfs-samba
```
> NOTE - Do note set a password on the `sshfs-samb` user.
{: .prompt-danger }

Become the sshfs-samba user.
```bash
su - sshfs-samba -s /bin/bash
```

Create an SSH key. Pres enter each time you are prompted taking the defaults.
```bash
ssh-keygen -t ed25519 -b 4096
```

Echo the new key in to Autohorized Keys and change the permissions.
```bash
echo ed25519.pub > ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
Edit the contents of the private key and copy it to save for the connection app. This will be needed to create the Mac App. The example uses nano, but you may use whatever method you prefer.
```bash
nano ~/.ssh/id_ed25519
```

It may be necessary to edit the ssh_config file to add the user to the `AllowUsers` sections.
```bash
nano /etc/ssh/sshd_config
```
Copy and paste the contents of the ed25519 into the script.


./dmgbuild 
Get info on the file paste the icon


/etc/sshd/sshd_config - Add the ssh user to the file 
	add to the end of the list of "Allow"

smbpasswd -a sshfs-samba user and set the password.
smbpasswd -e sshfs-samba (to enable)
