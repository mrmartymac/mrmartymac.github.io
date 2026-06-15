---
title: Setting up BLOX to Scoop API
date: 2026-04-30 08:48:05 -400
categories: [Documentation, Scoop7]
tags: [setup, blox, api] # TAG names should always be lowercase
author: mm
---
Configuring Scoop7 to receive articles from BLOX uses the `blox.php` script which is installed as part of the normal Scoop installation. The following instructions are points and steps that must be followed to complete the setup.

## Requests and Verification
1. Make a request to BLOX via the customer
```text
Could you please turn on the web services for our account and provide the key and secret for the API.
```
2. Request an export of the Sections from the BLOX site. They can be found in Settings -> Sections in BLOX. Request that they click the Export button and send the resulting file.
3. Ensure the `blox.php` script is in `/var/www/html/`
4. Make sure the URL you are going to use in BLOX for the webhook has a valid SSL certificate. If needed follow the instructions [here](http://martins-macbook-pro.local/posts/Install-ssl-cert-httpd/)
   
## Scoop Database Manager setup
Configure the Product within Scoop to have the API key, Secret, and Style Mappings.

![Product CMS tab](</images/blox-setup/BLOX-CMS Setup.png>)  
Use the **Body Content Template** to choose the order in which content is imported into the article body.  
![Body Content Template](</images/blox-setup/BLOX-Body Template.png>)  
The **Photo Caption Template** allows for the addition of extra characters and for the sequencing of the Caption and Credit for a photo.   
![Photo Caption Template](</images/blox-setup/BLOX-Photo Captiion Template.png>)  

Configure the **Template Map** to determine which Scoop template is applied to the article and into which Basket articles from each CMS Sections will be deposited.

It is possible to edit the Section export list you requested in step #2 above, but you need to know the ID values for the Templates and Baskets.

![Tempalte Mapp](</images/blox-setup/BLOX-Templates Map.png>)

## Content Manager Jobs
It is suggested to create three jobs in Content Manager. One to make a backup of the input files, one to import the input files, and the third to clean up the backups after a few days.

### Blox Import Backup Job
![Blox Import Backup Job](/images/blox-setup/BLOX-ImportBackup.png)  

### Blox Import Job
![Blox Import Job](/images/blox-setup/BLOX-Import.png)  

### Blox Import Backup Purge
![Blox Import Backup Purge](/images/blox-setup/BLOX-BackupPurge.png)