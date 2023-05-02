---
title: Diable Scoop Tasks 
date: 2023-05-02 10:52:18 -400
categories: [Cheatsheet, PostgreSQL]
tags: [disable, update, tasks] # TAG names should always be lowercase
author: mm
---
Connect to the postgres database and run the command shown below to disable active tasks.

Open PSQL
```sql
psql -U postgres
```

Connect to the Scoop database
```sql
\c scoop_db
```

Run query to disable active tasks.
```sql
UPDATE tasks
SET enabled = 'false'
WHERE enabled = 'true';
```

Disconnect from the database  
`\q`