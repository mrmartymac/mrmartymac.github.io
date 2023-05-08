---
title: Command to renew certificate
date: 2023-05-08 10:43:28 -500
categories: [Cheatsheet, Linux]
tags: [certificate, renew] # TAG names should always be lowercase
author: mm
---

```bash
openssl req -x509 -newkey rsa:4096 -sha512 -days 3650 -keyout /etc/xdg/SCS/Scoop.key \
    -nodes -out /etc/xdg/SCS/Scoop.crt \
    -subj "/C=US/ST=Pennsylvania/L=Nazareth/O=Software Consulting Services, LLC./CN=`hostname`"
chown scoop:scoop /etc/xdg/SCS/Scoop.{crt,key}
```