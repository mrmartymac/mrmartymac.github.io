---
title: Notes about install_scs.sh mirror setup
date: 2025-11-18 07:45:58 -400
categories: [Cheatsheets, Mirror]
tags: [mirror] # TAG names should always be lowercase
author: mm
---
## Information to collect before running
When running the script you will be asked to provide several pieces of information. They are listed below.

* Is this node the primary? [yes]:  
* Primary node [NodeName]:  
* Secondary node:  
* Secondary node IP:  
* Shared IP:  
* Shared Netmask:  
* Shared Broadcast:  
* Syno Logy Name:  

### Running the installation for `mirror`
```bash
./install_scs.sh mirror
```

If there are problems with the script completing all the prerequisites, run the command below to repeat just that part of the process.
```bash
/u/scs/bin/check_req.sh
```