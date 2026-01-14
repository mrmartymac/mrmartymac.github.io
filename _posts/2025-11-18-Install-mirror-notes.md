---
title: Notes about install_scs.sh mirror setup
date: 2025-11-18 07:45:58 -400
categories: [Cheatsheets, Mirror]
tags: [mirror] # TAG names should always be lowercase
author: mm
---

## Clone the Primary
Log in to the freshly cloned server.  
1. Change the passwords for users
2. Change the hostname
```bash
sudo hostnamectl set-hostname new-hostname
```
3. Reboot the server
```bash
reboot
```

## Modify the hosts file 
Edit `/etc/hosts` and enter the IP address and name(s) for the two servers.
```bash
nano /etc/hosts
```
## Configuring SSH Keys

1. Login and become root on the both servers.
2. Create an SSH key with the command.  
```bash
ssh-keygen -t ed25519
```
3. Display the key to copy to the other server.
```bash
cat /root/.ssh/id_ed25519.pub
```
4. Repeat steps two and three on the other server
6. Add the contents you noted in step 3 to `/root/.ssh/authorized_keys` on each server.

> The key from server "one" goes into the `authorized_keys` file on server "two". Then the key from server "two" goes into the `authorized_keys` file on server "one".
{: .prompt-info }

```bash
nano /root/.ssh/authorized_keys
```
You should now be able to SSH between servers as root.

### Configure default ssh port

1. Add the other server's pubic IP to the firewall with the following command where &lt;IP&gt; is the IP to whitelist: 
```bash
firewall-cmd --add-rich-rule="rule family=\"ipv4\" source address=\"<IP>\" service name=\"ssh-custom\" accept" --permanent
```

2. Repeat step 1 for any other IPs that need to be added (i.e. the site's IP).
3. Restart the firewall and ensure changes take effect. 
```bash
firewall-cmd --reload
```

4. Edit `/root/.ssh/config` and add the following lines: 
    1. Host &lt;OTHER\_HOST&gt;  
         Port &lt;SSH\_PORT&gt;
    2. Where &lt;OTHER\_HOST&gt; is the other server's hostname (i.e. tbr-news-media-2) and &lt;SSH\_PORT&gt; is the non-standard SSH port (i.e. 9322).
5.  Repeat the above steps on the other server.
```bash
nano /root/.ssh/config
```
## Information to collect before running
When running the script you will be asked to provide several pieces of information. They are listed below.

* Is this node the primary? [yes]:  
* Primary node [NodeName]:  
* Secondary node:  
* Secondary node IP:  
* Shared IP:  
* Shared Netmask:  
* Shared Broadcast:  
* Syno Logy Name:  

### Running the installation for `mirror`
```bash
./install_scs.sh mirror
```

If there are problems with the script completing all the prerequisites, run the command below to repeat just that part of the process.
```bash
/u/scs/bin/check_req.sh
```