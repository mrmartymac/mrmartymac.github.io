---
title: Attaching Linode Volume
date: 2026-04-02 09:19:55 -400
categories: [Cheatsheets, Linode]
tags: [linode, volume] # TAG names should always be lowercase
author: mm
---
Attaching a Volume to a Linode requires steps in the web UI as well as the command line.

Log into Linode and navigate the the Volumes panel. Locate the Volume and using any of the options Attach it to the desired Linode. Then `ssh` into the server and become `root` to run the commands below. You will need the connection information from the **Show Config** or **Volume Configuration**.

If the Volume was cloned from an existing Volume, you can skip this next step. Run the `mkfs.ext4` command as it is shown in the **Volume Configuration** panel.  

```bash
mkfs.ext4 "/dev/disk/by-id/<VolumeName>"
```

Once the volume has a filesystem, you can create a mountpoint for it. We usually do this as `/u` or `/ads` but we are considering other approaches. All you are doing is running a `mkdir` to create the directory that will become the location of the mounted Volume.

> This example uses `/u` as the directory in each command, replace it as needed.
{: .prompt-info }
```bash
mkdir /u
```

Now associate the Linode Volume with the directory you created in the previous step.
```bash
mount "/dev/disk/by-id/<VolumeName>" "/u"
```

If you want the volume to automatically mount every time your Linode boots, you’ll want to add a line like the following to your `/etc/fstab` file:
```bash
nano /etc/fstab
```
Add the following line. Also, ensure there is no other potentially conflicting mount in `fstab`
```bash
/dev/disk/by-id/<VolumeName> /u ext4 defaults,noatime,nofail 0 2
```

Reboot the server to ensure the volume is mounted as expected.
