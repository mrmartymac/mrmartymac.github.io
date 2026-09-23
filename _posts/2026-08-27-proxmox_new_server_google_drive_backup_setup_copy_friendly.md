---
title: Proxmox New Server Setup Guide
date: 2026-08-27 10:49:37 -400
categories: [Docmentation, Proxmox]
tags: [proxmox, rclone] # TAG names should always be lowercase
author: mm
---

## Google Drive Backup Mount and rclone VFS Cache

This guide documents the standard setup to follow when adding a new Proxmox server so that Google Drive backups behave consistently across all nodes.

> **Copy-friendly formatting:** Every command, command sequence, service file, configuration entry, and expected terminal output that you may need to reuse is shown in a fenced code block. Markdown viewers that provide code-block copy controls will therefore show a **Copy** button for those sections.

The goal is to ensure:

- Only one Google Drive rclone service is active.
- Google Drive is mounted at `/mnt/pve/gdrive-shared`.
- The rclone VFS cache does **not** use the Proxmox root filesystem.
- The cache has a dedicated local filesystem or a suitable dedicated local storage location.
- The rclone cache is capped so it cannot consume all available local storage.
- Proxmox backups can write to Google Drive without failing with `Broken pipe` or `no space left on device`.

---

# 1. Install rclone

On the new Proxmox node:

```bash
apt update
apt install -y rclone fuse3
```

Verify:

```bash
rclone version
```

---

# 2. Copy the Known-Good rclone Configuration

Create the configuration directory:

```bash
mkdir -p /root/.config/rclone
chmod 700 /root/.config/rclone
```

Copy the existing configuration from a working Proxmox node:

```bash
scp root@proxmox2:/root/.config/rclone/rclone.conf /root/.config/rclone/rclone.conf
```

Secure it:

```bash
chmod 600 /root/.config/rclone/rclone.conf
```

Verify the configured remotes:

```bash
rclone listremotes --config=/root/.config/rclone/rclone.conf
```

You should see:

```text
gdrive:
```

Test Google Drive access:

```bash
rclone lsd gdrive: --config=/root/.config/rclone/rclone.conf
```

---

# 3. Check for Old or Duplicate Google Drive Services

Before creating anything new, check for existing rclone processes and services:

```bash
ps auxww | grep '[r]clone'
```

```bash
systemctl --type=service --all | grep -iE 'rclone|gdrive'
```

```bash
systemctl list-unit-files | grep -iE 'rclone|gdrive'
```

Check existing mounts:

```bash
mount | grep -iE 'rclone|gdrive'
```

The desired final configuration is:

```text
Service:       rclone-gdrive.service
Google Drive:  gdrive:
Mount:         /mnt/pve/gdrive-shared
```

There should **not** be a second mount such as:

```text
/mnt/gdrive
```

and there should not be an obsolete service such as:

```text
gdrive.service
```

## Removing an obsolete `gdrive.service`

First make sure no backup is running:

```bash
ps aux | grep '[v]zdump'
```

Then:

```bash
systemctl disable --now gdrive.service
```

Verify:

```bash
systemctl is-enabled gdrive.service
systemctl is-active gdrive.service
```

The expected state is:

```text
disabled
inactive
```

Remove the old service:

```bash
rm /etc/systemd/system/gdrive.service
systemctl daemon-reload
systemctl reset-failed
```

Verify:

```bash
systemctl status gdrive.service
```

The expected result is:

```text
Unit gdrive.service could not be found.
```

---

# 4. Decide Where the rclone Cache Should Live

Do **not** allow the rclone VFS cache to use the default location under:

```text
/root/.cache/rclone
```

That places the cache on the Proxmox root filesystem and can fill `/` during large backups.

This previously caused backup failures such as:

```text
vma_queue_write: write error - Broken pipe
```

while rclone logged:

```text
no space left on device
```

Inspect the node:

```bash
df -hT
```

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

```bash
pvesm status
```

```bash
vgs
```

```bash
lvs -a -o lv_name,vg_name,lv_size,data_percent,metadata_percent,segtype
```

## Preferred cache choices

### Option A — Separate Local SSD

If the node has a separate local SSD with substantial free space, use a directory on that SSD.

Example:

```text
/mnt/pve/ssd-storage/rclone-cache
```

Create it:

```bash
mkdir -p /mnt/pve/ssd-storage/rclone-cache
chown root:root /mnt/pve/ssd-storage/rclone-cache
chmod 700 /mnt/pve/ssd-storage/rclone-cache
```

### Option B — Dedicated Thin LV

If there is no separate SSD, create a dedicated thin-provisioned LV.

A standard configuration is:

```text
150 GB cache filesystem
100 GB rclone cache limit
```

Create the thin LV:

```bash
lvcreate -V 150G -T pve/data -n rclone-cache
```

A thin-provisioning warning may appear. Review thin-pool utilization before proceeding.

Format it:

```bash
mkfs.ext4 -m 0 /dev/pve/rclone-cache
```

Create the mount point:

```bash
mkdir -p /var/lib/rclone-cache
```

Mount it:

```bash
mount /dev/pve/rclone-cache /var/lib/rclone-cache
```

Verify:

```bash
df -h /var/lib/rclone-cache
```

---

# 5. Make the Dedicated Cache LV Persistent

If using `/dev/pve/rclone-cache`, obtain its UUID:

```bash
blkid /dev/pve/rclone-cache
```

Edit:

```bash
nano /etc/fstab
```

Add:

```text
UUID=<actual-uuid> /var/lib/rclone-cache ext4 defaults 0 2
```

Test the entry:

```bash
umount /var/lib/rclone-cache
mount -a
```

Verify:

```bash
df -h /var/lib/rclone-cache
```

---

# 6. Create the Google Drive Mount Point

Create:

```bash
mkdir -p /mnt/pve/gdrive-shared
```

If the node is already part of the Proxmox cluster, check the cluster storage definition:

```bash
grep -A8 -B2 'gdrive-shared' /etc/pve/storage.cfg
```

Before mounting rclone, make sure the local directory does not contain files that would be hidden by the mount:

```bash
ls -la /mnt/pve/gdrive-shared
```

---

# 7. Create `rclone-gdrive.service`

Create:

```bash
nano /etc/systemd/system/rclone-gdrive.service
```

## Configuration when using `/var/lib/rclone-cache`

```ini
[Unit]
Description=Rclone Google Drive Mount
After=network-online.target
Wants=network-online.target
RequiresMountsFor=/var/lib/rclone-cache

[Service]
Type=notify
ExecStart=/usr/bin/rclone mount gdrive: /mnt/pve/gdrive-shared \
  --config=/root/.config/rclone/rclone.conf \
  --vfs-cache-mode writes \
  --cache-dir=/var/lib/rclone-cache \
  --vfs-cache-max-size=100G \
  --allow-other
ExecStop=/bin/umount -l /mnt/pve/gdrive-shared
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

## Configuration when using a separate SSD

For a cache directory such as:

```text
/mnt/pve/ssd-storage/rclone-cache
```

use:

```ini
[Unit]
Description=Rclone Google Drive Mount
After=network-online.target
Wants=network-online.target
RequiresMountsFor=/mnt/pve/ssd-storage

[Service]
Type=notify
ExecStart=/usr/bin/rclone mount gdrive: /mnt/pve/gdrive-shared \
  --config=/root/.config/rclone/rclone.conf \
  --vfs-cache-mode writes \
  --cache-dir=/mnt/pve/ssd-storage/rclone-cache \
  --vfs-cache-max-size=100G \
  --allow-other
ExecStop=/bin/umount -l /mnt/pve/gdrive-shared
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

---

# 8. Enable and Start rclone

Reload systemd:

```bash
systemctl daemon-reload
```

Enable and start the service:

```bash
systemctl enable --now rclone-gdrive.service
```

Check it:

```bash
systemctl status rclone-gdrive.service --no-pager -l
```

Verify the process:

```bash
ps auxww | grep '[r]clone'
```

There should be exactly one rclone mount process.

It should include:

```text
gdrive: /mnt/pve/gdrive-shared
--vfs-cache-mode writes
--cache-dir=...
--vfs-cache-max-size=100G
```

---

# 9. Verify the Mount

Run:

```bash
mount | grep -i gdrive
```

The desired result is a single mount:

```text
gdrive: on /mnt/pve/gdrive-shared type fuse.rclone
```

Check:

```bash
df -hT /mnt/pve/gdrive-shared
```

It should report:

```text
fuse.rclone
```

Check Google Drive contents:

```bash
ls -la /mnt/pve/gdrive-shared
```

```bash
ls -la /mnt/pve/gdrive-shared/ProxMoxBackups
```

---

# 10. Cluster Considerations

## Node Already in the Cluster

If the node is already part of the Proxmox cluster, the cluster-wide storage definition should already exist in:

```text
/etc/pve/storage.cfg
```

Check:

```bash
pvesm status | grep gdrive-shared
```

The storage should become active once `/mnt/pve/gdrive-shared` is mounted by rclone.

Do not create a duplicate Proxmox storage entry if `gdrive-shared` already exists cluster-wide.

## Node Not Yet in the Cluster

If the new node is **not yet part of the cluster**, configure the local pieces:

- rclone
- Google Drive credentials
- cache filesystem
- `/mnt/pve/gdrive-shared`
- `rclone-gdrive.service`

Then stop.

Do **not** create a separate standalone `gdrive-shared` Proxmox storage definition if the node will soon join an existing cluster.

After the node joins the cluster, verify:

```bash
pvesm status | grep gdrive-shared
```

The cluster's existing storage definition should appear automatically.

---

# 11. Final Configuration Check

Run:

```bash
echo "===== RCLONE PROCESSES ====="
ps auxww | grep '[r]clone'

echo
echo "===== SERVICES ====="
systemctl list-unit-files | grep -iE 'rclone|gdrive'

echo
echo "===== ACTIVE SERVICE ====="
systemctl status rclone-gdrive.service --no-pager -l

echo
echo "===== MOUNTS ====="
mount | grep -i gdrive

echo
echo "===== FILESYSTEMS ====="
df -h / /var/lib/rclone-cache /mnt/pve/gdrive-shared 2>/dev/null
```

The desired state is:

- Exactly one rclone process.
- Only `rclone-gdrive.service` enabled.
- Only `/mnt/pve/gdrive-shared` mounted from Google Drive.
- VFS cache located away from `/`.
- `--vfs-cache-max-size=100G` configured.
- Root filesystem usage does not increase substantially during backups.

---

# 12. Test a Backup

Choose a reasonably large VM.

In one SSH session, monitor the cache:

```bash
watch -n 5 '
echo "=== FILESYSTEMS ==="
df -h / /var/lib/rclone-cache
echo
echo "=== RCLONE CACHE ==="
du -sh /var/lib/rclone-cache 2>/dev/null
echo
echo "=== RCLONE STATUS ==="
systemctl status rclone-gdrive.service --no-pager | grep "vfs cache"
'
```

If using an SSD cache, substitute its cache path.

In another SSH session, run a manual backup:

```bash
vzdump <VMID> --storage gdrive-shared --mode snapshot --compress zstd
```

Watch for:

- The cache filesystem growing.
- `/` remaining essentially unchanged.
- No `no space left on device`.
- No `vma_queue_write: write error - Broken pipe`.

After the backup completes:

```bash
systemctl status rclone-gdrive.service --no-pager -l
```

It may temporarily show:

```text
uploading 1
```

That is normal. Proxmox has completed the backup while rclone continues uploading the cached file to Google Drive.

Monitor until the cache drains:

```bash
watch -n 10 '
df -h /var/lib/rclone-cache
echo
systemctl status rclone-gdrive.service --no-pager | grep "vfs cache"
'
```

Eventually the service should report:

```text
to upload 0, uploading 0
```

---

# 13. Common Failure Symptoms

## `vma_queue_write: write error - Broken pipe`

Check the rclone service immediately:

```bash
systemctl status rclone-gdrive.service --no-pager -l
```

Also check:

```bash
journalctl -u rclone-gdrive.service -n 100 --no-pager
```

A prior root cause was:

```text
no space left on device
```

under:

```text
/root/.cache/rclone/vfs/
```

This indicates that the VFS cache is still using the root filesystem or that the dedicated cache has filled.

## Google Drive Appears Active but Uses the Root Filesystem Size

If:

```bash
pvesm status
```

shows `gdrive-shared` with roughly the same capacity as `/`, check:

```bash
df -hT /mnt/pve/gdrive-shared
```

If it reports `ext4` rather than `fuse.rclone`, the rclone mount is not active and Proxmox is merely seeing the underlying local directory.

Restart:

```bash
systemctl restart rclone-gdrive.service
```

Then verify the mount again.

## Two Google Drive Mounts

If:

```bash
mount | grep -i gdrive
```

shows both:

```text
/mnt/gdrive
/mnt/pve/gdrive-shared
```

look for an obsolete:

```text
gdrive.service
```

Disable and remove it, keeping only:

```text
rclone-gdrive.service
```

---

# Standard Target Configuration

For future Proxmox nodes, the standard should be:

```text
Rclone remote:      gdrive:
Rclone service:     rclone-gdrive.service
Google Drive mount: /mnt/pve/gdrive-shared
VFS cache:          dedicated local storage
Cache filesystem:   150 GB when using a dedicated thin LV
Cache limit:        100 GB
Old gdrive.service: must not exist
Old /mnt/gdrive:    must not be mounted
```

The key lesson is that Proxmox can write backups directly to an rclone-mounted Google Drive, but `--vfs-cache-mode writes` requires enough local cache space. The cache should therefore never be left on the Proxmox root filesystem.
