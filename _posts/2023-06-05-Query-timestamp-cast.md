---
title: Query tasks using CASTS for time frame
date: 2023-06-05 10:17:20 -400
categories: [Cheatsheet, PostgreSQL]
tags: [casts, timestamp, task log] # TAG names should always be lowercase
author: mm
---
The query below is used to locate event logs where the number of exported articles is not zero.  Note the paramters of the task_id.

```sql
SELECT * FROM task_history 
WHERE task_id = 93 
    AND run_at > '20230527'::date 
    AND run_at < '20230602'::date 
    AND error_msg NOT LIKE '%exported 0%';
```