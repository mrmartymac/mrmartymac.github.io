---
title: Installing WordPress
date: 2023-03-21 09:04:29 -500
categories: [Installing Software]
tags: [wordpress] # TAG names should always be lowercase
author: mm
---
https://www.linuxtuto.com/how-to-install-wordpress-on-almalinux-9/

**WordPress** is a free, open-source and one of the most popular Content Management System (CMS) written in **PHP**, combined with **MySQL/MariaDB** database. Millions of websites are currently running on it and remains perhaps the most popular CMS for blogging, which was its original use case.

In this tutorial we’ll show you how to setup a [LAMP (Linux, Apache, MySQL/MariaDB, PHP)](https://www.linuxtuto.com/how-to-install-lamp-stack-on-almalinux-8-5/) stack and install WordPress on AlmaLinux 9 OS.

## Step 1: Update Operating System

Update your **AlmaLinux 9** operating system to make sure all existing packages are up to date:

```
sudo dnf update && sudo dnf upgrade -y
```

Also, install:

```
sudo dnf install nano wget unzip
```

## Step 2: Install Apache web server on AlmaLinux 9

You can install Apache via `dnf` package manager by executing the following command.

```
sudo dnf install httpd
```

You can start the `httpd` service and configure it to run on startup by entering the following commands:

```
sudo systemctl start httpd
sudo systemctl enable httpd
```

Verify the status of the `httpd` service using `systemctl status` command:

```
sudo systemctl status httpd
```

Output:

```
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
     Active: active (running)
       Docs: man:httpd.service(8)
   Main PID: 10643 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 213 (limit: 5738)
     Memory: 28.9M
        CPU: 198ms
     CGroup: /system.slice/httpd.service
             ├─10643 /usr/sbin/httpd -DFOREGROUND
             ├─10644 /usr/sbin/httpd -DFOREGROUND
             ├─10645 /usr/sbin/httpd -DFOREGROUND
             ├─10646 /usr/sbin/httpd -DFOREGROUND
             └─10647 /usr/sbin/httpd -DFOREGROUND
```

If `firewalld` is enabled consider allowing `HTTP` and `HTTPS` services:

```
sudo firewall-cmd --permanent --add-service={http,https}
sudo firewall-cmd --reload
```

You can test to make sure everything is working correctly by navigating to:

```
http://your-IP-address
```

If everything is configured properly, you should be greeted by the default Apache2 Page, as seen below.

![AlmaLinux 9 Test Page](https://i0.wp.com/www.linuxtuto.com/wp-content/uploads/2022/05/AlmaLinux_Test_Page.jpg?resize=897%2C427&ssl=1)

## Step 3: Install PHP and PHP extensions for WordPress

You can install PHP and other supporting packages using the following command:

```
sudo dnf install php php-curl php-bcmath php-gd php-soap php-zip php-curl php-mbstring php-mysqlnd php-gd php-xml php-intl php-zip
```

Verify if PHP is installed.

```
php -v
```

Output:

```
PHP 8.0.13 (cli) (built: Nov 16 2021 18:07:21) ( NTS gcc x86_64 )
Copyright (c) The PHP Group
Zend Engine v4.0.13, Copyright (c) Zend Technologies
with Zend OPcache v8.0.13, Copyright (c), by Zend Technologies
```

## Step 4: Install mysql and create a database

Install mysql client the following command:

```
sudo dnf install mysql
```

Install mysql server 

```
sudo dnf install mysql-server
```

Start the database server daemon, and also enable it to start automatically at the next boot with the following commands:

```
sudo systemctl start mysqld
sudo systemctl enable mysqld
```

Verify the status of the `MySQL` service using `systemctl status` command:

```
sudo systemctl status mysqld
```

Once the database server is installed, run the following command to secure your MariaDB server:

```
sudo mysql_secure_installation
```

Use the following options for the prompt.

```
Enter current password for root (enter for none): Just press the Enter
Set root password? [Y/n]: Y
New password: Enter your password
Re-enter new password: Repeat your password
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]:  Y
Reload privilege tables now? [Y/n]:  Y
```

Restart the database server for the changes to take effect.

```
sudo systemctl restart mysqld
```

Ensure Firewall is open
```
sudo firewall-cmd --zone=public --add-service=mysql --permanent
sudo firewall-cmd --reload
```

Login to mysql shell using the following command:

```
sudo  mysql -u root -p
```

To create a database, database user, and grant all privileges to the database user run the following commands:

```bash
CREATE DATABASE wordpress_db;
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'PaSSw0rd';
GRANT ALL ON wordpress_db.* TO 'wordpress_user'@'%';
FLUSH PRIVILEGES;
EXIT
```

## Step 5: Download WordPress on AlmaLinux 9

We will now download the latest version of WordPress from the [WordPress Official site](https://wordpress.org/download/).

Use the following command to download WordPress:

```
sudo wget https://wordpress.org/latest.zip
```

Extract file into the folder **/var/www/html/** with the following command,

```
sudo unzip latest.zip -d /var/www/html/
```

Change the permission of the website directory:

```
sudo chown -R apache:apache /var/www/html/wordpress/
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
```

## Step 6: Configure Apache Web Server for WordPress

Navigate to `/etc/httpd/conf.d` directory and run the following command to create a configuration file for your installation:

```
sudo nano /etc/httpd/conf.d/wordpress.conf
```

Add the following content:

```
<VirtualHost *:80>

ServerAdmin webmaster@your-domain.com

ServerName your-domain.com
ServerAlias www.your-domain.com
DocumentRoot /var/www/html/wordpress

<Directory /var/www/html/wordpress/>
        Options FollowSymlinks
        AllowOverride All
        Require all granted
</Directory>

ErrorLog /var/log/httpd/your-domain.com_error.log
CustomLog /var/log/httpd/your-domain.com_access.log combined

</VirtualHost>
```

Save the file and Exit.

Restart the Apache web server.

```
sudo systemctl restart httpd
```

## Step 7: Access WordPress Web Installer

Open your browser type your domain e.g `http://your-domain.com`  and click on the **`Let's go!`** button to complete the required steps:

![WordPress install Almalinux 9](https://i0.wp.com/www.linuxtuto.com/wp-content/uploads/2022/05/WordPress_Install.jpg?resize=897%2C427&ssl=1)

Provide the requested database information and click on the **`Submit`** button:

![WordPress install database](https://i0.wp.com/www.linuxtuto.com/wp-content/uploads/2022/05/wordpress-install-database.jpg?resize=897%2C427&ssl=1)

Start WordPress Installation by clicking on the **`Run the installation`** button:

![WordPress run the installation](https://i0.wp.com/www.linuxtuto.com/wp-content/uploads/2022/05/wordpress-run-installation.jpg?resize=897%2C257&ssl=1)

Provide the requested information and click on the **`Install WordPress`** button:

![WordPress Welcome Page](https://i0.wp.com/www.linuxtuto.com/wp-content/uploads/2022/05/wordpress-welcome-page.jpg?resize=897%2C512&ssl=1)

Once the WordPress is installed, enter your administrator user, password and click on the **`Log In`** button:

![WP Login Page](https://i0.wp.com/www.linuxtuto.com/wp-content/uploads/2022/05/wordpress-login.jpg?resize=897%2C391&ssl=1)

You will get the dashboard in the following screen:

![WordPress Dashboard](https://i0.wp.com/www.linuxtuto.com/wp-content/uploads/2022/05/wordpress-dashboard.jpg?resize=897%2C432&ssl=1)

## Comments and Conclusion

That’s it. You have successfully installed WordPress CMS (Content Management System) on AlmaLinux 9 OS.

If you have any questions please leave a comment below.


# If there is trouble with the version of php, needs to be > 8.0, use these steps.

## PHP 8.0 Installation on AlmaLinux or Rocky Linux

### Step 1. Add Remi Repository on your Linux

By default, the version of PHP available on our **AlmaLinux** or **Rocky** to install is **PHP 7.2**, so to get the latest version i.e **8.0** we need to add the third-party repository to get the wide range of PHP versions.

```
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

### Step 2: Run system update

To update the package repository so that our system could recognize the added newly repo, run the system update command:

```
sudo dnf update
```

### Step 3: Check available PHP module

Let’s check what are the available version of PHP on our AlmaLinux to install. For that run module list command to it using the DNF package manager.

```
sudo dnf module list php
```

You will see the PHP 8.0 version along with PHP 7.x to install in Remi’s Module repository for Enterprise Linux 8.

[![Check available php module in REMI repo on AlamLinux](https://www.how2shout.com/linux/wp-content/uploads/2021/06/Check-available-php-module-in-REMI-repo-on-AlamLinux.jpg "Check available php module in REMI repo on AlamLinux")](https://www.how2shout.com/linux/wp-content/uploads/2021/06/Check-available-php-module-in-REMI-repo-on-AlamLinux.jpg)

### Step 4: Enable php:remi-8.0 module

As there are multiple versions of PHP to install, thus we have to mention the version along with the repo name to enable the default source to get this scripting packages to install on our system. Here is the command to run but first reset the PHP module set for system repository packages.

```
sudo dnf module reset php
```

**And then run:**

```
sudo dnf module enable php:remi-8.0
```

**Note**: In case you want to enable some other module version of PHP, then just change the version number in the above-given command.

### Step 5: Command to install PHP 8.0 on AlmaLinux/Rocky from Remi Repo

Finally, run the command that will install the latest PHP on our Linux system.

```
sudo dnf install php -y
```

[![PHP 8.0 installation almalinux 8 or rocky](https://www.how2shout.com/linux/wp-content/uploads/2021/06/PHP-8.0-installation-almalinux-8-or-rocky.jpg "PHP 8.0 installation almalinux 8 or rocky")](https://www.how2shout.com/linux/wp-content/uploads/2021/06/PHP-8.0-installation-almalinux-8-or-rocky.jpg)

To install any particular extension, use the following syntax:

```
sudo dnf install php-extension-name
```

To check what are the installed and active extensions for your PHP, use:

```
php -m
```

### Step 5: Check Version

Once the installation is completed, let’s check what is the version finally we have on our system.

```
php -v
```

[![Check Version](https://www.how2shout.com/linux/wp-content/uploads/2021/06/Check-Version.jpg "Check Version")](https://www.how2shout.com/linux/wp-content/uploads/2021/06/Check-Version.jpg)


## Database commands for easy copy

```bash
CREATE DATABASE wordpress_db;
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'PaSSw0rd';
GRANT ALL ON wordpress_db.* TO 'wordpress_user'@'localhost';
FLUSH PRIVILEGES;
EXIT
```


## Fix for MySQL missing in PHP 

`sudo apt-get install phpmyadmin`
