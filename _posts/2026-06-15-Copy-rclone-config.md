---
title: Copy rclone config
date: 2026-06-15 14:56:23 -400
categories: [Documentation, ProxMox]
tags: [rclone, config] # TAG names should always be lowercase
author: mm
---


To ensure rclone is configured identically on both proxmox1 and proxmox2, you don't need to manually run through the rclone config interactive setup on the second node.

Instead, the cleanest and most reliable method is to configure it once on your first node and copy the configuration file directly to the second node.

Here is the step-by-step process to get them perfectly synced.


### 

---
Step 1: Locate the Configuration File on Node 1

Log into the shell of **proxmox1** (where your Google Drive is already working) and find where the configuration file is saved by running:


    Bash

rclone config file \


Typically, if you ran the configuration as root, it will be located at:

/root/.config/rclone/rclone.conf


### Step 2: Copy the Configuration to Node 2

Since your nodes are already clustered, they have secure SSH keys pre-shared between them. You can use scp to copy the file directly from **proxmox1** to **proxmox2**.

Run this command from the **proxmox1** shell:


    Bash

# Create the directory on node 2 just in case it doesn't exist yet \
ssh proxmox2 "mkdir -p /root/.config/rclone" \
 \
# Copy the configuration file over \
scp /root/.config/rclone/rclone.conf proxmox2:/root/.config/rclone/rclone.conf \



### Step 3: Verify the Remote on Node 2

Log into the shell of **proxmox2** and test that it can see the exact same remote using the copied credentials:


    Bash

# List the configured remotes (should match Node 1 exactly) \
rclone listremotes \
 \
# Test connectivity by listing the top-level directories of your Google Drive \
rclone lsd gdrive: \


*(Replace gdrive: with whatever name you originally gave your remote).*


### 

---
Step 4: Standardize the Mount Points

To make sure Proxmox treats this identically across the cluster, create the exact same mount directory on **both** nodes:


    Bash

# Run this on BOTH proxmox1 and proxmox2 \
mkdir -p /mnt/pve/gdrive-shared \



### Step 5: Automate the Mount with identical Systemd Services

To ensure the drive mounts automatically on boot and handles crashes gracefully, create an identical systemd service file on **both nodes**.

Create a new file at /etc/systemd/system/rclone-gdrive.service on both machines:


    Ini, TOML

[Unit] \
Description=Rclone Google Drive Mount \
After=network-online.target \
Wants=network-online.target \
 \
[Service] \
Type=notify \
ExecStart=/usr/bin/rclone mount gdrive: /mnt/pve/gdrive-shared \ \
  --config=/root/.config/rclone/rclone.conf \ \
  --vfs-cache-mode writes \ \
  --allow-other \
ExecStop=/bin/umount -l /mnt/pve/gdrive-shared \
Restart=on-failure \
User=root \
 \
[Install] \
WantedBy=multi-user.target \


Once the file is saved on both nodes, run the following commands on **both** instances to enable and start the drive:


    Bash

systemctl daemon-reload \
systemctl enable rclone-gdrive.service \
systemctl start rclone-gdrive.service \



### 

---
Step 6: Add it to the Proxmox Datacenter

Now that both nodes are actively mounting the drive to /mnt/pve/gdrive-shared, you can open your Proxmox web UI:



1. Go to **Datacenter** -> **Storage** -> **Add** -> **Directory**.
2. **ID:** gdrive
3. **Directory:** /mnt/pve/gdrive-shared
4. **Content:** Select Backup (and ISO image/Container template if desired).
5. **Nodes:** Leave this blank (default) or explicitly select both proxmox1 and proxmox2.

Because the underlying rclone config and mount points match exactly, both nodes will seamlessly read and write to the same Google Drive space.
