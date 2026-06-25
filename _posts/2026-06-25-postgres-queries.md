---
title: PostgreSQL Queries
date: 2026-06-25 11:47:30 -400
categories: [Cheatsheets, Postgresql]
tags: [postgres, postgresql, psql, queries,] # TAG names should always be lowercase
author: mm
---


### Bulk update of Groups for users
```sql
WITH new_groups (username, new_group) AS (
    VALUES
        ('mmacdonald', '_allbaskets'),
        ('ethan', '_allbaskets')
)
UPDATE users u
SET "group" = n.new_group
FROM new_groups n
WHERE u.name = n.username;
```