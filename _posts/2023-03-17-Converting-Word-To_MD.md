---
title: Converting Word document to Markdown using pandoc 
date: 2023-03-17 14:31:06 -400
categories: [cat1, cat2]
tags: [tag1, tag2] # TAG names should always be lowercase
author: mm
---
```bash
pandoc <fileName>.docx -t markdown --extract-media /Users/martinmacdonald/Documents/GitHub/mrmartymac.github.io/images/<docName>/ -s -o /Users/martinmacdonald/Documents/GitHub/mrmartymac.github.io/_posts/<docName>.md
```