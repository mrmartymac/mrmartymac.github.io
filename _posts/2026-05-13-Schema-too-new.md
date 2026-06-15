---
title: Schema too new error
date: 2026-05-13 14:53:50 -400
categories: [Cheatsheets, PostgreSQL]
tags: [schema, psql] # TAG names should always be lowercase
author: mm
---
These are the steps Ethan used to address the `schema too new error` error when migrating databases

```bash
journalctl -xu ScoopDaemon
```

> Schema version 66 is too new for this version!

```sql
psql -U postgres scoop_db
```
```sql
DELETE FROM migrations WHERE migration_version = 66;
DELETE FROM migrations WHERE migration_version = 65;
```
```bash
systemctl restart ScoopDaemon
systemctl status ScoopDaemon
```

