---
title: Migrate from KVM to ProxMox
date: 2026-06-17 09:49:00 -400
categories: [Documentation, ProxMox]
tags: [kvm, migrate, proxmox] # TAG names should always be lowercase
author: mm
---

Migrating virtual machines from an old **CentOS 7 server running `virt-manager` (KVM/QVM)** to **Proxmox VE (PVE)** is a fairly straightforward process. Since Proxmox uses KVM under the hood just like CentOS, the virtual disks are highly compatible.

Here is the step-by-step guide to executing the migration.

---

## Step 1: Prepare the VM on CentOS 7

Before moving anything, you need to clean up the guest operating system so it boots cleanly on Proxmox.

1. **Uninstall Guest Tools:** If the VM is running Windows, uninstall any Hyper-V or KVM-specific guest drivers that might conflict. For Linux VMs, standard kernels usually handle the hardware change automatically.
2. **Note the VM Settings:** Take note of the VM's resources (vCPUs, RAM size, and especially the network configuration/MAC address if it uses static IPs).
3. **Shut Down the VM:** Ensure the VM is fully powered off on the CentOS 7 host:  

```bash
virsh shutdown <vm_name>
```



---

## Step 2: Locate and Transfer the Virtual Disk

You need to copy the underlying storage file from CentOS to your Proxmox node.

1. **Find the disk image path:** On CentOS, look up the XML configuration to find where the disk image (`.qcow2` or `.raw`) lives:  

```bash
virsh domblklist <vm_name>
```


*(Usually, they are located in `/var/lib/libvirt/images/`)*.
2. **Transfer the disk to Proxmox:** Use `scp` or `rsync` to copy the disk file over to a temporary directory on your Proxmox server (e.g., `/root/`):  

```bash
rsync -avz /var/lib/libvirt/images/your-disk.qcow2 root@<proxmox_ip>:/root/
```

> Becaue of space limitations on ProxMox2 it may be necessary to copy the disk to the Synology to then import. To do that use this command.
{: .prompt-info }

```bash
rsync -avz /vm-images/support_servers/<VM Name>.qcow2 root@10.0.0.52:/mnt/pve/Synology03/
```


---

## Step 3: Create a Shell VM on Proxmox

Next, you need to create a "container" VM on Proxmox that will hold this disk.

1. In the Proxmox Web UI, click **Create VM** in the top right.
2. **General:** Assign a VM ID (e.g., `107`) and a name.
3. **OS:** Select **Do not use any media**.
4. **System:** Leave defaults (or select Qemu Agent if you plan to install it later).
5. **Disks:** **Delete the default created disk.** (Click the trash icon next to it). We will be importing your existing disk instead.
6. **CPU / Memory:** Match or exceed the resource allocations from your old CentOS server.
7. **Network:** Choose your bridge (usually `vmbr0`).
8. Finish the wizard without starting the VM.

### Notes on virt-man

To get the resources of the VM on virt-man run
```bash
virsh dominfo <vm_name>
```


---

## Step 4: Import the Disk into Proxmox

Now, use Proxmox's native CLI tool to import and attach the CentOS disk to the new VM shell.

1. Log in to your Proxmox server via SSH or open the node's **Shell** from the web UI.
2. Run the `qm importdisk` command. This will automatically convert (if necessary) and move the disk to your Proxmox storage pool:  

```bash
qm importdisk <VM_ID> <PathYouCopiedVMto>.qcow2 <storage_pool>
```


* *Example:* `qm importdisk 107 /root/your-disk.qcow2 local-lvm`



---

## Step 5: Attach and Configure the Disk in Proxmox

Once the import finishes, the disk will appear in the Proxmox UI, but it won't be plugged into the VM yet.

1. Go to your new VM in the Proxmox Web UI and navigate to the **Hardware** tab.
2. You will see an **Unused Disk 0**. Select it and click **Edit** (or double-click it).
3. Select the bus type:
* **SCSI** (with VirtIO SCSI controller) is recommended for best performance on Linux/Windows.
* If the VM fails to boot due to driver issues, change this to **IDE** or **SATA** temporarily until you can install VirtIO drivers.


4. Go to the **Options** tab -> **Boot Order**.
5. Click **Edit**, check the box for your newly attached disk, and drag it to the top of the boot priority list.

---

## Step 6: Power On and Post-Migration Cleanup

1. **Start the VM** from the Proxmox UI and open the **Console** to monitor the boot process.
2. **Network Adjustment:** Because Proxmox presents a different virtual network card (usually VirtIO or Intel E1000) than `virt-manager`, the MAC address and interface name (e.g., changing from `eth0` to `ens18`) will likely change. Log into the guest OS and update your network configuration files if it doesn't grab a DHCP lease immediately.
3. **Install Guest Tools:**  

* **Linux:** Install `qemu-guest-agent`  

```bash
dnf install qemu-guest-agent
```
* **Windows:** Download and mount the [Fedora VirtIO ISO](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers) to install the missing optimized network and ballooning drivers.