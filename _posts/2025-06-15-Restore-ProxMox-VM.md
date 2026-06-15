---
title: Restore ProxMox VM from Google Drive
date: 2026-06-15 12:50:08 -400
categories: [Documentation, ProxMox]
tags: [backup, restore, vm] # TAG names should always be lowercase
author: mm
---

Assuming the SCS instance is the current instance, *proxmox1* is configured to connect to Google Drive using the **Shared Drives/IT Information/ProxMoxBackups** folder. The backup can be copied within the cluster using `scp`. If the backup files are coming from another location, all that is needed is to copy the backup files, usually a `.zst` file, to `/var/lib/vz/dump` on the ProxMox node to which you want to restore the VM.

As of the writing of this document, using the IP address of the ProxMox instance is suggested for the `scp` command.

The scp command to copy the file from proxmox1 to another ProxMox node in the cluster follows.
```bash
scp root@10.0.0.50:/mnt/gdrive/ProxMoxBackups/dump/<TargetBackup>.vma.zst /var/lib/vz/dump/
```

Once the file copies over locally to the desitnation ProxMox instance, it will instantly appear in your GUI! 

1. You can click local (proxmox-name)
2. Backups, and you will see the file sitting there.
3. You can then simply click it and hit Restore right from the graphical interface.

![ProxMox Restore](/images/proxmox/ProxMox-Restore1.png)


## Troubleshooting 

It happened the first time that I did this, the backup I restored had been completed with a virtual CD/DVD attached to the instance. This caused the startup to fail. The solution follows.

1. In your Proxmox sidebar, click directly on your newly restored VM:.
2. In the middle menu, click on the Hardware tab.
3. Look down the list for your CD/DVD Drive (ide2). You will see it pointing to that missing AlmaLinux path.
4. Click on the CD/DVD Drive line to highlight it, then click the Edit button at the top of the hardware panel.
5. Change the setting to Do not use any media (or choose a local ISO if you actually need one), and click OK.

![ProxMox - eject CD/DVD](/images/proxmox/ProxMox-Restore2.png)