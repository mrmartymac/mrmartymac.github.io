---
title: SMB from Windows 11 or Windows Server 2025
date: 2025-06-27 14:06:12 -400
categories: [Cheatsheet, Windows]
tags: [windows, smb] # TAG names should always be lowercase
author: mm
---


## Resolve SMB / Windows Explorer issue
On Windows Server 2025 I was encountering an issue where I could ping a server but could not access the SMB share through Windows Explorer. The error message I was seeing read `An extended error has occurred.` The resolution was the Powershell command below.
```bash
Set-SmbClientConfiguration -RequireSecuritySignature $false
```