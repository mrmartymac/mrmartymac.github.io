---
title: Configuring ProxMox to use Google Drive
date: 2026-05-29 14:44:18 -400
categories: [Documentation, ProxMox]
tags: [proxmox, google drive] # TAG names should always be lowercase
author: mm
---
I used the following sites for setting up `rclone` on ProxMox to connect Google Drive.
[Synbricks]([URL](https://syncbricks.com/proxmox-google-drive-integration-rclone-setup-guide/))
[Rclone](https://rclone.org/drive/#making-your-own-client-id)
[Google Console](https://console.cloud.google.com/))


## Troubleshooting with AI

QUESTION: after running the "rclone mount synbricks: /mnt/gdrive --allow-other --cache-db-purge --poll-interval 10m" command, the prompt just sits there with no reply.

ANSWER: Don't panic—this is completely normal behavior!

When you run the `rclone mount` command, it executes in the **foreground** by default. It isn't frozen; it is actively holding that terminal session open to maintain the connection and keep your Google Drive mounted.

If you close that terminal window or press `Ctrl + C`, the drive will unmount.

Here is how you can handle this:

### 1. Test it right now (Open a second terminal)

To verify it's working without disturbing the current process:

* Open a **new, separate SSH terminal window** to your Proxmox server.
* Run the following commands to see if your files are there:  
  
```bash
cd /mnt/gdrive
ls
```

If you see your Google Drive folders, the mount is successful!

---

### 2. The solution for permanent use (Background Service)

You don't want to keep a terminal window open forever just to keep your drive mounted. The guide you are following actually addresses this in **Step 6**.

To make the mount run silently in the background and survive system reboots, you should stop your current frozen-looking command with **`Ctrl + C`** and set up the systemd service:

1. **Create the service file:**  
```bash
nano /etc/systemd/system/gdrive.service
```


2. **Paste the configuration** provided in the guide:  

```ini
[Unit]
Description=rclone for G drive for synbricks
AssertPathIsDirectory=/mnt/gdrive
After=network.service

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount --config=/root/.config/rclone/rclone.conf synbricks: /mnt/gdrive --allow-other --cache-db-purge --poll-interval 10m
ExecStop=/bin/fusermount -u /mnt/gdrive
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```


3. **Save and exit** (Press `Ctrl + O`, `Enter`, then `Ctrl + X`).  
4. **Enable and start it:**  
```bash
systemctl enable gdrive.service
systemctl start gdrive.service
```



This will run the exact same mount command in the background, freeing up your command prompt instantly and ensuring it starts automatically every time Proxmox boots up.

QUESTION: running systemctl start results in this error: Warning: The unit file, source configuration file or drop-ins of gdrive.service changed on disk. Run 'systemctl daemon-reload' to reload units.
Assertion failed on job for gdrive.service.

ANSWER: That warning and error combination usually happens for two specific reasons: you edited the file after trying to start it, and the directory where you are trying to mount Google Drive does not exist yet.

Here is how to fix it in two quick steps:

### 1. Fix the "changed on disk" warning

Whenever you create or modify a service file in `/etc/systemd/system/`, `systemd` needs to refresh its internal cache. Run this command to clear the warning:  

```bash
systemctl daemon-reload
```

---

### 2. Fix the "Assertion failed" error

If you look closely at the configuration from the [SyncBricks guide](https://syncbricks.com/proxmox-google-drive-integration-rclone-setup-guide/), the service file includes this line:
`AssertPathIsDirectory=/mnt/gdrive`

This means the service will intentionally crash and refuse to start if the exact folder `/mnt/gdrive` does not exist on your Proxmox local storage.

Run this command to ensure the target directory is created:  

```bash
mkdir -p /mnt/gdrive
```

---

### 3. Restart the service

Now that the warning is cleared and the directory exists, try starting the service again:  

```bash
systemctl start gdrive.service
```

To confirm that it is finally running perfectly in the background, check its status:  

```bash
systemctl status gdrive.service
```

You should see a green **`active (running)`** status!

QUESTION: The result after following these steps is now: Assertion failed on job for gdrive.service.

ANSWER: That specific "Assertion failed" error means `systemd` checked the conditions required to run your script and found something it didn't like.

Since we already confirmed the folder `/mnt/gdrive` exists, the failure is almost always caused by a **stray, hidden rclone process** still hanging onto that folder from your previous manual testing, causing the systemd asset check to fail.

Let's clear out the ghost processes and reset the mount. Run these commands in order:

### 1. Force unmount the directory

Even if it looks empty, a failed or interrupted rclone attempt might be locking the mount point. Force it to release:  

```bash
fusermount -u /mnt/gdrive
```

*(If it says "invalid argument" or "not mounted", that's fine—it means it was already clean).*

### 2. Kill any hidden rclone processes

Make absolutely sure there isn't a background rclone process lingering from your first manual command:  

```bash
killall rclone
```

*(If it says "no process found", you can safely move to the next step).*

### 3. Clear systemd's failed state and restart

Systemd sometimes gets stubborn and remembers the "failed" state until you explicitly tell it to reset. Run this sequence to reload, clear the error, and start fresh:  

```bash
systemctl daemon-reload
systemctl reset-failed gdrive.service
systemctl start gdrive.service
```

---

### Still failing? Let's check the exact error log

If it throws the assertion error again after running those three steps, systemd will tell us exactly *why* it failed in the system logs. Run this command to look at the real-time log:  

```bash
journalctl -u gdrive.service -n 20 --no-pager
```

**What to look for in the log:**

* If you see **`rclone: command not found`**: It means rclone isn't located at `/usr/bin/rclone`. Run `which rclone` to find its real path, and update the `ExecStart` line in your `/etc/systemd/system/gdrive.service` file.
* If you see **`Config file ... not found`**: Double-check that your rclone configuration is actually saved under the root user profile (`/root/.config/rclone/rclone.conf`).