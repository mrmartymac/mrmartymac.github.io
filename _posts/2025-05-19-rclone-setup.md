---
title: Working with rclone
date: 2025-05-19 14:39:12 -400
categories: [Cheatsheets, rclone]
tags: [rclone, object, storage, proxmox, backup] # TAG names should always be lowercase
author: mm
---

# **Proxmox backups**

Nightly at 01:00 a backup of the Proxmox VM is made and placed in the `cd /var/lib/vz/dump` of the Proxmox server. The backup job is configured to keep three days of backups due to space. 

However, the backups will be **moved** to the Linode Object Storage bucket called **scs-alere2025-01-bakups** at 02:00 every night. 

There is a cron job configured to handle this job nightly. A screenshot of the backup job is below. The command in the crontab entry is:

Command run in the crontab entry
```bash
rclone move /var/lib/vz/dump linode:scs-alere2025-01-bakups
```

Crontab entry
```bash
 0 2 * * * rclone move /var/lib/vz/dump linode:scs-alere2025-01-bakups
 ```

# **Preparing to use S3 Backup Storage**

## **Create Object Storage Bucket**

On Linode you need to log in and navigate to Storage →Object Storage →Buckets and create the bucket for the backup. After creating the bucket, create the Access Key you will need during the configuration of Rclone.

Creating the Access Key is a little odd in the writers opinion. Follow these steps to create your key.

1. Click **Create Access Key**  
2. Enter a label that describes the key.  
3. When clicking in the **Regions** field, choose **Select All**.   
4. Set the **Limited Access** switch to **ON** so the switch is **blue.**  
5. On the top row, click the **No Access** under the **Select All** in the Region column.  
6. Now, find your newly created Bucket in the list and adjust the permissions to that bucket accordingly.  
7. Click Create Access Key at the bottom.  
8. Copy the **Access Key** and **Secret Key**.  
9. Click the **I Have Saved My Secret Key**.

## **Install Rclone**

I installed rclone on Proxmox to facilitate the upload of backups to a Linode Bucket.

`apt-get install rclone`

### **Configure Rclone**

Following the instructions on this [website](https://rclone.org/s3/#linode), I configured rclone to work with Linode. It all starts by running the command `rclone config`

#### Editing the configuration manually

The configuration file for rclone can be edited using the following command.

`nano /root/.config/rclone.rclone.conf`

After much trial and error I found that editing the configuration file helpful and the example below has the **region** and the **location\_constraint** left blank. The normal config process puts some values in there that cause errors. For obvious security rea

### **Testing Rclone**

The backups created by Proxmox are stored in `/var/lib/vz/dump` on the ProxMox node. Depending on the type of backup, the file extensions vary. There are also secondary files containing the logs and notes related to the backup.

#### Helpful Rclone commands

* `rclone –help`  
* `rclone copy –help`  
* `rclone copy <source:sourcepath> <dest:destpath> –dry-run`  
* `rclone copy <source:sourcepath> <dest:destpath> -P (shows progress)`  
* `rclone move <source:sourcepath> <dest:destpath> -P (shows progress)`


