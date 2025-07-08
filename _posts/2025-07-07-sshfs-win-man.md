---
title: Setting up SSH-Win-Manager
date: 2025-07-08 14:10:42 -400
categories: [Cheatsheet, SSHFS-Win-Manager]
tags: [ssh, sshfs] # TAG names should always be lowercase
author: mm
---
## Setting up SSHFS-Win-Manager

1. Install SSHFS-Win-Manager [[link](https://github.com/evsar3/sshfs-win-manager/releases)]
2. Create a file in `C:\users\<UserName>\.ssh` with a name of `scoopid`
3. Enter the contents of the Private Key. Ensuring the line enders are UNIX LF and not CR|LF.
4. Ensure there is one blank line at the file.
 ### Example of the Private Key
 ![Private Key Screenshot](/images/sshfs-win-man/ssh-key.png)

 ### Connection setup
 ![Connection Setup](/images/sshfs-win-man/ssh-win-man-setup.png)