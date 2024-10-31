---
title: Configuring New Linode
date: 2024-10-02 10:42:27 -400
categories: [Documentation, Linode]
tags: [setup, server, linode, firewall] # TAG names should always be lowercase
author: mm
---
After starting the new Linode follow these steps to finish the setup.
## Disable the following

Disable SELinux by setting “SELINUX=disabled” in /etc/selinux/config
```bash
nano /etc/selinux/config
```

Disable spectre mitigation
```bash
grubby --args=’nospectre_v2 notpi’ --update-kernel=ALL
```

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
firewall-cmd --add-port=9006/tcp --permanent
firewall-cmd --add-port=9008/tcp --permanent
```
Add Scoop if relevant:
```bash
firewall-cmd --add-port=7267-7268/tcp --permanent
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
firewall-cmd --add-port=9006/tcp --permanent
firewall-cmd --add-port=9008/tcp --permanent
firewall-cmd --add-port=7267-7268/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all
```

## Install kits needed for other sections.
```bash
dnf install -y epel-release
dnf install -y fail2ban fail2ban-systemd
dnf --enablerepo=powertools install -y fuse-sshfs x2goserver
dfn install tar
dnf install perl-libwww-perl
dnf install ncftp
dnf install httpd
dnf install postgresql
dnf install bzip2
dnf install lbzip2
```

## Configuring fail2ban
```bash
dnf install -y epel-release
dnf install -y fail2ban fail2ban-systemd
cd /etc/fail2ban
cp jail.conf jail.local
nano jail.local
```

### Modify jail.local:
1. Uncomment the `#ignoreip = ... `line
2. Add 10.0.0.0/24 for SCS connections
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
systemctl enable x2gocleansessions.service
systemctl restart x2gocleansessions.service
```
Make sure there are no issues with `systemctl status x2gocleansessions.service`

Install xterm
```bash
dnf install -y xterm
```

## Nagios setup

Become root on the relevant server
Install tar, make a directory in /root/apps and cd into /root/apps/nagios and download the nagios script using cURL, make get_nagios.sh executable.
```bash
mkdir -p /root/apps/nagios
cd /root/apps/nagios
curl -f -k -O https://scssupport.com/scs_ms/nagios/get_nagios.sh
chmod +x get_nagios.sh
./get_nagios.sh
```

Set the host name.
```bash
hostnamectl set-hostname <NameOfServer>
```
Configure the hostname in the `/usr/local/scs_ms/nagios/etc/server_settings.conf` by setting the `HOST_NAME=` variable to the host name of the server.
```bash
nano /usr/local/scs_ms/nagios/etc/server_settings.conf
```

Run `/usr/local/scs_ms/nagios/bin/run_tests.sh` to test the nagios scripts
```bash
/usr/local/scs_ms/nagios/bin/run_tests.sh
```

Add the line below to crontab:
```
*/15 * * * * /usr/local/scs_ms/nagios/bin/run_tests.sh > /dev/null 2>&1
```

```bash
crontab -e
```


## CAS Framework install

Become the root user
```bash
cd /u/updates
```
Download step1.sh from the ethan directory on the SCS FTP
```bash
chmod +x step1.sh
./step1.sh
```

## Change SSH port

Edit the `/etc/ssh/sshd_config` file with your preferred text editor.
```bash
nano /etc/ssh/sshd_config
```

Find the line that has "#port 22" and un-comment the line, then change 22 to the port you wish to use.  
Change:  
#port 22  
To:  
port <NewPortNumber>  

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

## Getting SSL Cert for CAS

Follow instructions at https://certbot.eff.org/instructions
Below is an example for Apache + CentOS8/RHEL8
```bash
dnf install epel-release
dnf upgrade -y
```
Ensure that /etc/yum.repos.d/epel.repo has enabled=1 under the first heading
```bash
dnf install snapd -y
```
After this, reset /etc/yum.repos.d/epel.repo if you modified it
```bash
systemctl enable --now snapd.socket
ln -s /var/lib/snapd/snap /snap
```
Exit and relogin as root to refresh the snap installation
```bash
snap install core && snap refresh core
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot
certbot certonly --webroot -w /var/www/html
```
For the email, use “certs@newspapersystems.com”
Type “Y” for the Terms of Use
Type “N” for the email offer
Type the URL for the site you’re certifying
```bash
cd /etc/letsencrypt/live/<URL>
cat cert.pem chain.pem privkey.pem > /etc/pki/tls/certs/<URL>.pem
```
Check for le_check_cert.sh in /u/scs/tools/bin and le_check_cert.conf.default in /u/scs
```bash
ll /u/scs/tools/bin/le*
```

If they don’t exist, you can grab them from the ftp (currently in ethan/)
```bash
cp /u/scs/le_check_cert.conf.default /u/scs/le_check_cert.conf
```
Replace the variable CAS_SITE_DOMAIN with the proper URL (<URL>)
Replace the variable CERT_FILE with the proper pem file (<URL>.pem)
Set up the cron to run this script every day ( 0 3 * * * /u/scs/tools/bin/le_check_cert.sh )
```bash
crontab -e
```
```bash
cd /etc/haproxy
mv haproxy.cfg haproxy.old
```
Copy haproxy.cfg.default from the ftp (currently in ethan/)
Replace <URL> and rename the file to haproxy.cfg
If you’re not on CentOS7, you may need to add “session_id” to the end of the last line (under backend bk_ws)
```bash
cd /etc/httpd/conf
nano httpd.conf
``` 
Modify the line “Listen 80” to be “Listen 8080”
If you’re on CentOS8 (not sure about higher versions):cd 
```bash
cd /etc/httpd/conf.modules.d
```
Modify 00-mpm.conf such that:
The “LoadModule mpm_prefork…” line is UNCOMMENTED
All other “LoadModule…” lines are COMMENTED
```bash
systemctl restart httpd && systemctl restart haproxy
```

