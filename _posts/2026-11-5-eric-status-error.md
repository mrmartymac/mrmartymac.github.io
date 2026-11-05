---
title: 
date: 2026-11-05 08:32:35 -400
categories: [Cheatsheets, ERIC]
tags: [eric, reboot, update] # TAG names should always be lowercase
author: mm
---
After running updates it is sometimes needed to check the ERIC status.

To check that the service is running
```bash
systemctl status scs_eric_web_api
```

To ensure the process is present
```bash
ps -efwww | grep eric_input
```
