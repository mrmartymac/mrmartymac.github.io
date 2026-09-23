---
title: Linode CLI Commands
date: 2026-08-20 08:54:51 -400
categories: [Cheatsheets, Linode]
tags: [cli, linode] # TAG names should always be lowercase
author: mm
---

To suppress the default formatted table and output a plain list, use the `--text` or `--no-headers` options along with `--format`:

* **Plain list with headers (tab-separated text):**
```bash
linode-cli linodes list --tags <your-tag-name> --text

```


* **Plain list of just the labels (one per line):**
```bash
linode-cli linodes list --tags <your-tag-name> --format label --text --no-headers

```


* **Plain list of custom columns (e.g., label and IP address):**
```bash
linode-cli linodes list --tags <your-tag-name> --format label,ipv4 --text --no-headers

```