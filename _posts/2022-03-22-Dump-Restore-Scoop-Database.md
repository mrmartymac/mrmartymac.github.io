---
title: Dump and Restore a Scoop database
date: 2023-03-22 14:25:24 -400
categories: [Cheatsheet, PostgreSQL]
tags: [postgres, psql, database, dump, restore, scoop_db] # TAG names should always be lowercase
author: mm
---

Dump the database 
```
pg_dump -U postgres scoop_db > Scoop.dump
```
psql-16 version
```
/usr/pgsql-16/bin/pg_dump -U postgres scoop_db > Scoop.dump
```

Transfer the dump

Stop Scoop
```
systemctl stop ScoopDaemon
```

Drop the current DB
```
dropdb -U postgres scoop_db
```

Open PSQL
```
psql -U postgres
```

Create new Scoop DB
```
CREATE DATABASE scoop_db;
```
To exit PSQL type:  

`\q`

Restore the database
```
psql -U postgres -d scoop_db -f scoop.dump
```

Start Scoop
```
systemctl start ScoopDaemon
```