---
title: Windows Helpers
date: 2024-11-15 08:08:18 -400
categories: [Windows, Cheatsheets]
tags: [helper, iis, iso, .net35] # TAG names should always be lowercase
author: mm
---
## Installing .NET 3.5 when install media is not available.

1. Download the Evaluation ISO of the server version in question from the Microsoft Evaluation Downloads.
2. Mount the ISO
3. In Server Manager Add the Feature for .NET 3.5 and specify the location by capturing the path to the mounted ISO, `\Sources\sxs` folder.

![Capture Path](../images/windows-helpers/NET35-1.png)
![Server Manager](../images/windows-helpers/NET35-1.png)