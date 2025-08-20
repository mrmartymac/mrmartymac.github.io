---
title: How to Setup Let's Encrypt SSL Certificate
date: 2025-08-20 13:55:18 -400
categories: [Installation, WordPress]
tags: [ssl, certificate, cert, security] # TAG names should always be lowercase
author: mm
---

[How to Setup Let's Encrypt SSL Certificate with Apache on AlmaLinux 10](#How+to+Setup+Let%27s+Encrypt+SSL+Certificate+with+Apache+on+AlmaLinux+10)

* * *

Let's Encrypt is a non-profit certificate authority run by the Internet Security Research Group that provides free SSL/TLS certificates. In this guide, we will install and configure Let's Encrypt SSL on AlmaLinux 10.

## [Pre-requisites](#Pre-requisites)

* * *

*   `root` user access to the server.
*   `Apache` Web server installed and running.
*   You may refer to our [LAMP Stack Installation on AlmaLinux 10](https://wiki.crowncloud.net/index.php?How_to_Install_LAMP_Stack_on_AlmaLinux_10) if it's not already installed.

## [Install EPEL Repository and mod\_ssl](#Install+EPEL+Repository+and+mod_ssl)

* * *

```
dnf install epel-release mod_ssl -y
```


## [Install Certbot](#Install+Certbot)

* * *

```
dnf install certbot python3-certbot-apache -y
```


## [Configure Apache Virtual Host](#Configure+Apache+Virtual+Host)

* * *

> Example domain: `blog.domainhere.info` — replace this with your actual domain name.

Create a new Apache configuration file:

```
nano /etc/httpd/conf.d/blog.domainhere.info.conf
```


Add the following configuration:

```
<VirtualHost *:80>
    ServerName blog.domainhere.info
    ServerAlias blog.domainhere.info
    DocumentRoot /var/www/html

    <Directory /var/www/html/>
        Options -Indexes +FollowSymLinks
        AllowOverride All
    </Directory>

    ErrorLog /var/log/httpd/blog.domainhere.info-error.log
    CustomLog /var/log/httpd/blog.domainhere.info-access.log combined
</VirtualHost>
```


Save and exit.

## [Restart Apache and Check Status](#Restart+Apache+and+Check+Status)

* * *

```
systemctl restart httpd
systemctl status httpd
```


## [Configure Firewall](#Configure+Firewall)

* * *

```
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```


## [Generate Let's Encrypt SSL Certificate](#Generate+Let%27s+Encrypt+SSL+Certificate)

* * *

```
certbot --apache
```


Follow the on-screen instructions. Example:

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

Which names would you like to activate HTTPS for?
1: blog.domainhere.info

Select the appropriate number: 1
Requesting a certificate for blog.domainhere.info

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/blog.domainhere.info/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/blog.domainhere.info/privkey.pem
```


Certbot will also configure your Apache SSL automatically.

## [Verify SSL Installation](#Verify+SSL+Installation)

* * *

Open your browser and visit:

```
https://blog.domainhere.info
```


You should see a lock icon, confirming that SSL is successfully installed and HTTPS is working.

> Congratulations! You have successfully installed Let's Encrypt SSL Certificate with Apache on AlmaLinux 10.