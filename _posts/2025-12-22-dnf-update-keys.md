---
title: DNF Update - Key change issue
date: 2025-12-22 08:24:21 -400
categories: [Cheatsheets, Updates]
tags: [dnf, key, mismatch, gpg] # TAG names should always be lowercase
author: mm
---

## When running `dnf update` you may encounter this issue
![DNF Update Key Issue](/images/dnf-update/DNFUpdateKeyIssue.png)

To correct it run the following command from the [website]([URL](https://almalinux.org/blog/2023-12-20-almalinux-8-key-update/))] 

```bash
rpm --import https://repo.almalinux.org/almalinux/RPM-GPG-KEY-AlmaLinux
```

