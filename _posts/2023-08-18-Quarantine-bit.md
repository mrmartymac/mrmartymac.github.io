---
title: Fix Quarantine bit on downloaded files
date: 2023-08-18 13:31:33 -400
categories: [Cheatsheet, Mac]
tags: [fix, quarantine, zip, downloaded] # TAG names should always be lowercase
author: mm
---

If so, your "Scoop URL Handler" might have the "com.apple.quarantine" "extended attribute," which I believe is what produces the "is damaged and can't be opened" alert. You can check this by opening "Terminal," "cd"-ing to wherever your "Scoop URL Handler" resides, then:

`xattr "Scoop URL Handler.app"`  
com.apple.FinderInfo  
com.apple.quarantine

If "com.apple.quarantine" indeed appears, clear it with:

`xattr -cr "Scoop URL Handler.app"`  

where "c" means clear all extended attributes, and "r" means recursively to all files within.