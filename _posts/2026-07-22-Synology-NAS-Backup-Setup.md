---
title: Backup Linode Object Storage to Synology
date: 2026-07-22 08:30:47 -400
categories: [Cheatsheets, Synology]
tags: [backup, s3, linode, synology, cloud] # TAG names should always be lowercase
author: mm
---

I have a Synology NAS I would like to use to backup Object Storage buckets from Linode to. What is the best way to get the sync to start and run nightly? 

Worked for 12s 

# **Recommended approach: Synology Cloud Sync** 

For the simplest, most maintainable solution, use Synology’s **Cloud Sync** package with Linode configured as an **S3-compatible custom provider** . Linode Object Storage exposes an Amazon S3-compatible API, and Cloud Sync supports S3-compatible services, one-way downloads, scheduling, integrity checks, and bandwidth controls. Akamai TechDo… +2 



## **1. Create a dedicated Linode Object Storage key** 

In the Akamai/Linode Cloud Manager: 

**1.** Open **Object Storage** . 

**2.** Create a new access key specifically for the NAS. 

**3.** Restrict it to the buckets being backed up, preferably with **read-only** access. 

**4.** Record: 

   - Access key 

   - Secret key 

   - Bucket name 

   - Cluster/region endpoint 

The endpoint normally resembles: 

```
us-east-1.linodeobjects.com
```

Use the actual endpoint shown for your bucket. Linode documents that each Object Storage region has an S3 endpoint hostname. Akamai TechDo… 

A read-only key is strongly preferable because the NAS should not need permission to alter or delete the source bucket. 

## **2. Install Cloud Sync** 

On the Synology: 

**1.** Open **Package Center** . 

**2.** Install **Cloud Sync** . 

**3.** Open Cloud Sync. 

**4.** Create a new connection. 

**5.** Select **S3 Storage** or **Amazon S3-compatible storage** . 

### **6.** Choose the option for a custom server or custom endpoint. 

Enter: 

```
Server address: https://us-east-1.linodeobjects.com
Access key:     your Linode access key
Secret key:     your Linode secret key
```

Do not include the bucket name in the server address unless Cloud Sync specifically requests it separately. 

## **3. Configure a download-only task** 

Configure the task as: 

```
Remote path:  Linode bucket or bucket folder
Local path:   /volume1/Linode-Object-Storage/<bucket-name>
Sync direction: Download remote changes only
```

For multiple buckets, I would create a separate connection or task for each one: 

```
/volume1/Linode-Object-Storage/customer-images
/volume1/Linode-Object-Storage/site-backups
/volume1/Linode-Object-Storage/document-archives
```

Enable the advanced consistency or integrity check if available. Cloud Sync can compare file hashes to improve verification of synchronized files. Synology 

## **4. Set the nightly schedule** 

Cloud Sync’s schedule works by defining when synchronization is allowed. For example, allow it to run between: 

```
Start: 1:00 AM
Stop:  6:00 AM
Every day
```

Cloud Sync will check and download changes during that window. It is not necessarily a single batch job that starts at exactly 1:00 AM; it operates during the permitted schedule window. 

# **Important backup-safety decision** 

A pure mirror is not necessarily a backup. If someone deletes an object from Linode and the deletion is reproduced on the NAS, you have not preserved the deleted object. 

I recommend both of these protections: 

## **Preserve destination files** 

When configuring Cloud Sync, look for an option resembling: 

```
Do not remove files in the destination folder when they are removed in the source
```

Enable it when available. The exact wording can differ by Cloud Sync version. 

That gives you **copy-like behavior** rather than a strict mirror. 

## **Enable Synology snapshots or versioning** 

Place the destination inside a Btrfs shared folder and enable **Snapshot Replication** , perhaps with: 

```
Hourly snapshots: retain 24
Daily snapshots:  retain 30
Weekly snapshots: retain 12
Monthly snapshots: retain 12
```

This protects against: 

Accidental deletion in Linode 

- Corrupted or overwritten objects 

- Cloud Sync propagating an unwanted change Ransomware or accidental changes on the NAS 

# **Alternative: rclone scheduled through Task Scheduler** 

For more control, particularly guaranteed copy-not-delete behavior and detailed logs, use `rclone` . Rclone has a specific Linode S3 provider configuration. Rclone +1 

A typical command would be: 

Bash `rclone copy \ linode:my-bucket \ /volume1/Linode-Object-Storage/my-bucket \ --create-empty-src-dirs \ --fast-list \ --transfers 4 \ --checkers 8 \ --log-file /volume1/Linode-Object-Storage/logs/my-bucket.log \ --log-level INFO` 

Use `copy` , not `sync` : 

- `rclone copy` downloads new and changed objects but does not remove existing 

- destination files. 

`rclone sync` makes the destination match the bucket and can delete local files. 

You could run that nightly from: 

```
Control Panel
  → Task Scheduler
  → Create
  → Scheduled Task
  → User-defined script
```

Synology recommends using absolute paths, placing scripts and logs on shared volumes rather than the system partition, and explicitly invoking the shell. Synology Know… +1 

For example: 

Bash 

```
/bin/bash /volume1/scripts/backup-linode-buckets.sh
```

# **My recommendation** 

Start with **Cloud Sync configured as download-only** , with destination deletion disabled and Btrfs snapshots enabled. It is integrated into DSM, has visible status and history, and does not require maintaining a third-party binary. 

Use **rclone** instead when you need: 

- A true once-nightly job 

- More detailed logging 

- Explicit copy-without-deletion behavior 

- Exclusion rules 

- Multiple bucket automation in one script 

- Email or other custom failure handling 

For a long-term backup system rather than simple synchronization, **rclone** `copy` **plus Synology snapshots** is the strongest of the two arrangements. 

How would I configure Cloud Sync to connect to Linode Object Storage? 

Here is the configuration I would use in **Synology Cloud Sync** . 

# **1. Create a Linode access key** 

In **Akamai Cloud Manager** : 

**1.** Open **Object Storage** . 

**2.** Select **Access Keys** . 

**3.** Click **Create Access Key** . 

**4.** Give it a descriptive name, such as: 

```
Synology Cloud Sync
```

**5.** Restrict it to the region and bucket you intend to download. 

**6.** Assign **Read Only** permission to that bucket. 

**7.** Copy both values immediately: 

   - Access Key 

   - Secret Key 

Linode displays the secret when the key is created, so store it securely. Linode 

recommends explicitly including the applicable region when creating restricted Object Storage keys. Akamai TechDo… +2 

# **2. Find the bucket’s S3 endpoint** 

In Cloud Manager, open your Object Storage bucket and locate its **S3 endpoint** or **hostname** . 

It will look similar to one of these: 

```
us-east-1.linodeobjects.com
```

or, for newer regional endpoints, something similar to: 

```
us-mia-1.linodeobjects.com
```

Use the exact endpoint shown for your bucket. Linode specifically recommends using the regional Amazon S3 endpoint hostname when configuring third-party S3 clients. Akamai TechDo… +1 



Do not use the bucket’s public website URL. You need the regional **S3 API endpoint** . 

# **3. Install Cloud Sync** 

On the Synology: 

**1.** Open **Package Center** . 

**2.** Search for **Cloud Sync** . 

**3.** Install and open it. 

**4.** Click the **+** button to add a connection. 

# **4. Select the S3 provider** 

Choose: 

```
S3 Storage
```

Depending on your DSM and Cloud Sync version, the option may instead be labeled: 

```
Amazon S3 Compatible Cloud
```

or: 

```
S3 Compatible
```

Cloud Sync supports Amazon S3-compatible cloud services, which is appropriate because Linode Object Storage uses an S3-compatible API. Akamai TechDo… +1 

# **5. Enter the Linode connection information** 

Choose the custom or third-party provider option and enter: 

|**Cloud Sync field**|**Value**|
|---|---|
|Service provider|Custom server URL / S3 compatible|
|Server address|Your Linode regional endpoint|
|Access key|Linode access key|
|Secret key|Linode secret key|



For example: 

|`Server address: us-ea`|`st-1.linodeobjects.com`|
|---|---|
|`Access key:     YOUR_`|`ACCESS_KEY`|
|`Secret key:     YOUR_`|`SECRET_KEY`|



Start with the hostname **without** `https://` if Cloud Sync presents a field specifically labeled **Server address** . Synology’s S3-compatible instructions commonly expect the server hostname in that field. 

If your version instead explicitly requests a **Server URL** or **Endpoint URL** , use: 

```
https://us-east-1.linodeobjects.com
```

Do not add: 

```
/my-bucket-name
```

to the endpoint. Cloud Sync should retrieve the bucket list separately. 

# **6. Select the bucket** 

After Cloud Sync authenticates, it should present a list of buckets accessible with that key. 

Select the desired Linode bucket. 

If no buckets appear, the most common causes are: 

- The key does not include the bucket’s region. 

- The key lacks permission to list or read the bucket. 

- The wrong regional endpoint was entered. 

- The endpoint includes the bucket name when Cloud Sync expects only the hostname. The access key and secret key do not belong together. 

# **7. Create the synchronization task** 

Configure the task approximately like this: 

```
Connection name:
Linode - Bucket Name
Local path:
/volume1/Linode-Backups/bucket-name
Remote path:
/
Sync direction:
Download remote changes only
```

The important setting is: 

```
Download remote changes only
```

That prevents Cloud Sync from uploading local NAS changes back into Linode. 

# **8. Review advanced settings** 

I recommend: 

```
Data encryption: Disabled
File conversion: Disabled
Schedule: Enabled
```

Cloud Sync’s client-side encryption is generally intended for files being uploaded to cloud storage. Since you are downloading an existing bucket, enabling it would not encrypt the existing Linode objects in place and may prevent the local files from being directly usable. 

Also review any option resembling: 

```
Do not remove files in the destination folder
when they are removed from the source
```

Enable it when available. That makes the NAS copy more useful as a backup rather than a strict mirror. 

Be aware that Cloud Sync’s exact deletion options vary by provider and DSM version. Protecting the destination shared folder with Synology snapshots remains the safer approach. 

# **9. Configure the nightly schedule** 

Open the task’s schedule settings and allow synchronization during a nightly window, for example: 

```
Every day
1:00 AM through 6:00 AM
```

Cloud Sync schedules generally control the periods during which synchronization is allowed. It may continue checking for changes throughout that window rather than performing exactly one single pass at 1:00 AM. 

# **10. Test it** 

Before relying on the task: 

**1.** Place a small test object in the Linode bucket. 

**2.** Run or resume the Cloud Sync task. 

**3.** Confirm the file appears under: 

```
/volume1/Linode-Backups/bucket-name
```

**4.** Modify the test object in Linode. 

**5.** Confirm the updated object downloads. 

**6.** Delete the test object from Linode. 

### **7.** Confirm whether the NAS copy is retained or deleted. 

That final deletion test is important because it tells you whether your particular Cloud Sync configuration is acting as a **backup copy** or a **mirror** . 

A solid final arrangement would be: 

```
Linode Object Storage
        ↓
Cloud Sync — download only
        ↓
Btrfs shared folder
        ↓
Snapshot Replication
```

This gives you the current bucket contents plus recoverable historical NAS snapshots. 

