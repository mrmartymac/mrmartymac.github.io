---
title: Linode-cli adding tags
date: 2026-06-16 08:00:51 -400
categories: [Cheatsheets, Linode]
tags: [cli, tags, linode] # TAG names should always be lowercase
author: mm
---

The following are the options for using the `linode-cli` to add tags to Linode resources

```bash
# Linode
linode-cli linodes update 123 --tags prod --tags web

# Volume
linode-cli volumes update 456 --tags prod --tags database

# NodeBalancer
linode-cli nodebalancers update 789 --tags prod --tags frontend

# Domain
linode-cli domains update 12345 --tags customer-a

# Firewall
linode-cli firewalls update 67890 --tags prod --tags security
```