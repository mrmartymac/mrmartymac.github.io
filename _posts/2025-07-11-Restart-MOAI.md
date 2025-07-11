---
title: Restarting MOAI
date: 2025-07-11 08:03:06 -400
categories: [Cheatsheets, MOAI]
tags: [moai, restart] # TAG names should always be lowercase
author: mm
---
## Steps to restart MOAI when CAS is having issues.

Change directories to the MOAI Server direcotry.
```bash
cd /u/moai/server/
```

Check the status of MOAI
```bash
./moai.py status
```

If the status is not "OK", restart MOAI
```bash
./moai.py restart
```

Check the status again to make sure everything is running. The result should resemble this.
![MOAI Status](/images/moai/MOAI-status.png)