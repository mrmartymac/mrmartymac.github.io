---
title: ProxMox Cluster Failure
date: 2026-08-10 13:35:13 -400
categories: [Documentation, ProxMox]
tags: [proxmox, cluster] # TAG names should always be lowercase
author: mm
---

It is completely understandable to pause and figure out *why* this happened before moving forward. Cluster issues in Proxmox can be deeply frustrating, especially when a single node failure causes cascading issues across remaining nodes.

Here is a breakdown of why this failure chain occurred:


### 

---
1. The Core Root Cause: Corosync Quorum Loss

Proxmox relies on **Corosync** to manage cluster state. By default, a Proxmox cluster requires **strict majority consensus (> 50% of total votes)** to perform any write or sync operations on the cluster filesystem (/etc/pve).



* You started with **4 expected votes**: proxmox1, proxmox2, proxmox3, and proxmox04.
* To achieve quorum, the cluster needed **at least 3 active votes** ($\lfloor 4/2 \rfloor + 1 = 3$).
* When proxmox3 failed permanently and proxmox2 went offline simultaneously, active votes dropped to **2 out of 4**.


### 

---
2. Why Web Shells & Commands Broke

Once quorum was lost, Proxmox automatically put /etc/pve into **read-only safety mode** to prevent split-brain data corruption.

Because /etc/pve was locked:



* **Web UI Shells (termproxy) failed:** Web shells connect between nodes using cluster-synced SSH host keys located in /etc/pve/nodes/&lt;node>/ssh_known_hosts. When /etc/pve locked up, key validation and terminal ticket allocation failed with exit code 1.
* **Commands hung:** Service restarts (systemctl restart pveproxy pvedaemon) and cluster commands (pvecm updatecerts) hung indefinitely waiting for a read/write lock on the filesystem that could not be granted without quorum.


### 

---
3. Why Even-Numbered Clusters (4 Nodes) Are Vulnerable

A 4-node cluster is inherently more vulnerable than a 3-node or 5-node cluster:



* **3 Nodes:** Can lose 1 node and maintain quorum ($2/3 = 66\%$).
* **4 Nodes:** Can lose 1 node ($3/4 = 75\%$), but losing a **second** node drops it to $50\%$, immediately breaking quorum.
* **Result:** A 4-node cluster has the exact same fault tolerance to node failures as a 3-node cluster, but with an extra point of failure.


### 

---
How to Prevent This in the Future

Once proxmox2 is rejoined, you will have a **3-node cluster** (proxmox1, proxmox2, proxmox04), which naturally tolerates 1 node failure without losing quorum ($2/3$ votes active).

To prevent future lockups, consider these architectural improvements:



1. **Add a QDevice (Quorum Device):** \
If you ever drop to 2 nodes or want extra redundancy for a 3-node cluster, set up a **Proxmox External Vote Image (corosync-qnetd)** on an external lightweight device (a Raspberry Pi, a NAS, or a tiny low-power Linux VM outside the cluster). It acts as a tie-breaker vote so a single host going down never stalls the remaining cluster.
2. **Understand pvecm expected for Emergencies:** \
If a host ever goes down unexpectedly and locks the UI again, your immediate first command via direct SSH should always be: \
Bash \
pvecm expected &lt;number_of_currently_online_nodes> \
 \
This instantly forces quorum on surviving nodes and stops web proxy and SSH services from hanging.