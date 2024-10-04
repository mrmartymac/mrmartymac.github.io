---
title: Creating SCS Admin user
date: 2024-10-01 15:33:46 -400
categories: [Cheatsheet, Linux]
tags: [user, admin] # TAG names should always be lowercase
author: mm
---

Creating SCS admin User
```bash
groupadd scs
useradd -g scs -c "scs admin user" -s 	bin	bash -d 	u	users_scs	scsadmin -u 7000 -m scsadmin
usermod -aG wheel scsadmin
passwd scsadmin
```
