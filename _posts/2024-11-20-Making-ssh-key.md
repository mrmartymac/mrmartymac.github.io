---
title: Making ssh key
date: 2024-11-20 14:48:58 -400
categories: [Cheatsheet, Linux]
tags: [ssh, key, security] # TAG names should always be lowercase
author: mm
---

## Create the key on the local machine
```bash
ssh-keygen
```

Alternately if an RSA key is required
```bash
ssh-keygen -t rsa
```

Enter a passphrase when prompted


Copy the key to the server
```bash
ssh-copy-id username@remote_host
```

Ensure the permissions are correct. 
Home dir must be 700
.ssh folder must be 700
authorized_keys file must be 600

[Reference document](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)