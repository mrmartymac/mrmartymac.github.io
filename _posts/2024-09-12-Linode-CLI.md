---
title: Linode CLI
date: 2024-09-12 08:10:03 -400
categories: [Cheatsheet, Linode]
tags: [linode, cli] # TAG names should always be lowercase
author: mm
---

## Linode CLI

Get invoice into CSV
```
linode-cli account invoice-items 20122307 --text >> Linode-20211221.csv   
```

Collect Volume information in JSON, Markdown, Text/CSV
```
linode-cli volumes list --pretty
linode-cli volumes list --markdown
linode-cli volumes list --text
```