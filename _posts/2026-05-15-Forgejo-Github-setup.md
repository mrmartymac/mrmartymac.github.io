---
title: Create Forgejo instance for GitHub sync on ProxMox
date: 2026-05-15 14:06:16 -400
categories: [Documentation, Gitea]
tags: [github, gitea, proxmox, forgejo] # TAG names should always be lowercase
author: mm
---

The best way to host a Git instance on Proxmox for GitHub backups is to use a **Forgejo (or Gitea)** instance running in an **LXC (Linux Container)**. This setup is significantly more resource-efficient than a full Virtual Machine and provides built-in "Mirroring" features that automate the backup process.

### 1. Choose Your Software

While there are several options, **Forgejo** (a community-driven fork of Gitea) is currently the gold standard for homelabbers in 2026 due to its lightweight nature and native GitHub Actions compatibility.

* **Forgejo / Gitea:** Best for personal backups. Extremely fast, uses < 200MB of RAM, and includes a "Migration" tool specifically for mirroring GitHub repos.
* **GitLab:** Overkill for simple backups. It requires at least 4GB (ideally 8GB) of RAM and is much more complex to manage.

---

### 2. Recommended Deployment: LXC Container

Using an **LXC** instead of a VM allows you to share the host's kernel, resulting in near-zero overhead.

**The "Easy Way" (Proxmox Helper Scripts):**
The most efficient way to deploy this is using the [Proxmox VE Helper-Scripts](https://tteck.github.io/Proxmox/).

1. Open your Proxmox Node Shell.
2. Run the Forgejo LXC script:  

```bash  
bash -c "$(curl -fsSL https://github.com/community-scripts/ProxmoxVE/raw/main/ct/forgejo.sh)"

```


3. Follow the prompts. Use the "Default" settings for a quick setup, or "Advanced" if you want to specify a static IP or larger disk size.

---

### 3. Automating the GitHub Backup (Mirroring)

Once Forgejo is running, you don't need to manually `git pull` every day. You can set up a **Pull Mirror**:

1. **Generate a GitHub Token:** Go to your GitHub settings and create a **Personal Access Token (Classic)** with `repo` scopes.
2. **Create a New Migration:** In your new Forgejo instance, click the **+** icon in the top right and select **New Migration**.
3. **Select GitHub:** Choose GitHub as the source.
4. **Configure the Mirror:**
* Paste your GitHub Repository URL.
* Provide your GitHub username and the Token you just created.
* **Crucial:** Check the box that says **"This repository will be a mirror"**.


5. **Sync:** Forgejo will now automatically sync your repository every 8 hours (default), ensuring your local copy is always up to date.

---

### 4. Comparison of Hosting Methods on Proxmox

| Feature | LXC Container (Recommended) | Virtual Machine (VM) | Docker (inside VM/LXC) |
| --- | --- | --- | --- |
| **Performance** | Native speed, lowest overhead | High overhead | Dependent on host |
| **RAM Usage** | ~150MB - 512MB | ~2GB+ (OS overhead) | ~200MB + Docker overhead |
| **Backups** | Fast (Proxmox Backup Server) | Standard | Easy to move volumes |
| **Complexity** | Simple (single script) | Moderate | Moderate |

### Pro-Tip: Storage and Backups

* **Proxmox Backup Server (PBS):** If you aren't using it yet, point your Proxmox instance to a PBS datastore. It will perform incremental, deduplicated backups of your Git LXC, making your "backup of a backup" rock solid.
* **Bind Mounts:** If you have a large NAS (like TrueNAS), you can [bind mount](https://www.google.com/search?q=https://pve.proxmox.com/wiki/Linux_Container%23_bind_mount_points) a network share directly into the LXC so your Git data is stored on your redundant array rather than just the Proxmox boot drive.

Do you have a specific number of repositories you're looking to mirror, or are you planning to back up an entire GitHub Organization?

