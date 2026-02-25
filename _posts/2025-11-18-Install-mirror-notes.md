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

## Add PRIVATE IP to both servers
In the Linode dashboard, add a PRIVATE IP4 address to each server. Use this as the PRIMARY and SECONDARY IP address in the hosts file. Also when configuring mirroring.

## Network configuration

1. On the secondary, become root.
2. Run the `nmtui` command:
```bash
nmtui
```
3. Select **Edit a connection** and press **Enter**.
4. Highlight the main Ethernet device, select **Edit...**, then press **Enter**.
5. Next to **IPv4 CONNECTION**, select **Show**, then press **Enter**.
6. Next to **Addresses**, select **Add...** and press **Enter**.
7. Set the address to be the main IP located at the top of the Linode dashboard.  
The dashboard reads **Public IP Addresses**
8. Select **Add...** to add another address.
9. Enter the private IPv4 address.
10. Down below, next to **IPv6 CONNECTION**, select **Automatic...**, then press **Enter**.
11. Select **Ignore** and press **Enter**.
12. At the bottom, select **OK** and press **Enter**.
13. At the bottom of the **Ethernet** screen, select **Back** and press **Enter**
14. Back at the entry screen of **nmtui**, select **Set system hostname** and press **Enter**.
15. Set the hostname to be the same as the Linode name.
16. Select **OK** and press **Enter**.
17. Close **nmtui** and reboot the Linode **OR** use the command below.
```bash
systemctl restart NetworkManager
```
![NMTUI](/images/mirror_notes/nmtui.png)
18. Ensure you can still connect.
19. Install pip3 via
```bash
dnf install -y python3-pip
```
1.  Install the linode-cli tool via 
```bash
pip3 install linode-cli --upgrade
```
1.  In the top right of any Linode page, click on your username, then **API Tokens**.
2.  Click **Create A Personal Access Token**.
3.  Set the label to the name of the site (i.e. tbr-news-media).
4.  Set the expiry to **Never**.
5.  Set everything to **No Access** except **IPs**, **Linodes**, and **Volumes**.
6.  Click **Create Token**.
7.  Run the `linode-cli` command. 
```bash
linode-cli
```
    1. Paste your personal access token in.
    2. Accept all the defaults.
1.  Run `linode-cli volumes list` and take note of the id for the volume you created prior.
```bash
linode-cli volumes list
```
1.  Modify /etc/hosts and add both servers using their private IPs.
2.  Create a directory in root's home directory called **apps**.
3.  Create a directory in the **apps** directory called **linode**.
4.  Grab a copy of the **limkprime.sh** script and place it in the **linode** folder.
```bash
ncftpget -u ftpaccess -p knuckle ftp.newspapersystems.com . ethan/limkprime.sh
```
1.  Modify the **limkprime.sh** script with the following changes: 
```bash
nano limkprime.sh
```
    1. Set **this\_linode\_id** to be the Linode's id for this server. 
        1. This can be found on the Linode dashboard at the bottom of the top table or in the URL.
    2. Set **other\_node\_name** to be the other node's hostname (i.e. tbr-news-media-1).
    3. Set **shared\_ip** to be the new shared IPv4 address.
    4. Set **volume\_id** to the previously noted volume id.
    5. Set **mount\_name** to the provided volume mount name (i.e. /dev/disk/by-id/scsi-0Linode\_Volume\_tbr-news-media-ads)
    6. Set **region** to be the Linode's region. 
        1. You can get the region by looking at the **LISH Console via SSH** parameter on the Linode's dashboard. It follows after **lish-** and before **.linode.com**.
1.  Run 
```bash
./limkprime.sh all
```
> The options of `all`, `volumes`, or `ip` exist
{: .prompt-tip }
1.  Ensure the volume was mounted with `df`. You should see the /ads mount.
2.  Repeat the above on the primary.
3.  If everything works, the primary server should have the /ads volume mounted along with the shared IP assigned.

## Modify the hosts file 
Edit `/etc/hosts` and enter the IP address and name(s) for the two servers.
```bash
nano /etc/hosts
```

```bash
192.168.x.x PrimaryServeName PrimaryServerName.newspapersystems.com
192.168.x.x SecondaryServeName SecondaryServerName.newspapersystems.com
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

> The key from server **one** goes into the `authorized_keys` file on server **two**. Then the key from server **two** goes into the `authorized_keys` file on server **one**.
{: .prompt-info }

```bash
nano /root/.ssh/authorized_keys
```
You should now be able to SSH between servers as root.

### Configure default ssh port

1. Add the other server's **pubic** amd **private** IP addresses to the firewall with the following command where &lt;IP&gt; is the IP to whitelist: 
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