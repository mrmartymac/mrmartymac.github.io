---
title: WebEdit Notes
date: 2024-05-07 00:01:15 -400
categories: [Cheatsheets, WebEdit]
tags: [link, symbolic] # TAG names should always be lowercase
author: mm
---
## Creating symbolic link between versions of WebEdit
Create a symbolic link between versions in the WebEdit folder.
```shell
cd /usr/share/webedit
```

The link command will allow the older version to server as the version that matches the current server version number.

```shell
ln -s <oldVersion> <newVersion>
```
