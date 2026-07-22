---
title: Forgejo for Jekyll Docs
date: 2026-07-09 14:28:57 -400
categories: [cat1, cat2]
tags: [tag1, tag2] # TAG names should always be lowercase
author: mm
---

## Clean up old CT if needed
```bash
pct stop 200
pct destroy 200
```

Then recreate it using discovery instead of hard-coded template names:
```bash
pveam update

CTID=200
HOSTNAME="forgejo-docs"
TEMPLATE_STORAGE="local"
ROOTFS_STORAGE="local-lvm"
BRIDGE="vmbr0"

TEMPLATE="$(pveam available --section system | awk '/debian-12-standard/ {print $2}' | tail -n 1)"

echo "Using template: $TEMPLATE"

pveam download "$TEMPLATE_STORAGE" "$TEMPLATE"

pct create "$CTID" "$TEMPLATE_STORAGE:vztmpl/$TEMPLATE" \
  --hostname "$HOSTNAME" \
  --cores 2 \
  --memory 2048 \
  --swap 512 \
  --rootfs "$ROOTFS_STORAGE:20" \
  --net0 name=eth0,bridge="$BRIDGE",ip=dhcp \
  --features nesting=1,keyctl=1 \
  --unprivileged 1 \
  --onboot 1

pct start "$CTID"
pct enter "$CTID"
```

Inside the container
```bash
apt update
apt full-upgrade -y
apt install -y sudo git curl wget unzip vim rsync ca-certificates gnupg ruby-full build-essential
```

Then clone your repo:
```bash
cd /opt
git clone YOUR_GITHUB_REPO_URL forgejo-homelab
cd forgejo-homelab
```
DEsign notes
```txt
cat > ROADMAP.md <<'EOF'
# Forgejo Homelab Roadmap

## Design Principles

1. Never hard-code what the system can discover.
2. Every script should explain what it is doing.
3. Scripts should be safe to re-run when possible.
4. Each phase should end with a working checkpoint.

## Phase 1
- Create clean Proxmox LXC
- Install base Debian tools
- Clone toolkit repo

## Phase 2
- Install Caddy
- Serve local docs folder

## Phase 3
- Install Forgejo

## Phase 4
- Add Jekyll build workflow

## Phase 5
- Add backups and maintenance scripts
EOF

cat > TODO.md <<'EOF'
# TODO

- [ ] Create clean LXC
- [ ] Install base packages
- [ ] Clone repo
- [ ] Install Caddy
- [ ] Install Forgejo
- [ ] Install Jekyll
- [ ] Configure Forgejo Actions
- [ ] Publish first docs site
EOF
```