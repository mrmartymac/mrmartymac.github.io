---
title: eLibrary information
date: 2023-03-09 14:28:40 -500
categories: [Cheatsheet, eLibrary]
tags: [elibrary] # TAG names should always be lowercase
author: mm
---
# Database configuration

## Create databse
Open PSQL
```
psql -U postgres
```

```sql
CREATE USER elibrary;
CREATE DATABASE elibrary_db OWNER elibrary ENCODING = 'UTF8' LC_COLLATE = 'en_US.UTF-8' LC_CTYPE = 'en_US.UTF-8';
```
## Create tables in the database

```bash
cat <instalDir>/elibrary.sql | sudo -i -u postgres psql -U elibrary elibrary_db
```
## Assign owner to elibrary_db tables if incorrect

```sql
ALTER TABLE article_pdfs OWNER TO elibrary;
ALTER TABLE article_texts OWNER TO elibrary;
ALTER TABLE articles OWNER TO elibrary;
ALTER TABLE doc_pdfs OWNER TO elibrary;
ALTER TABLE fullpage_pictures OWNER TO elibrary;
ALTER TABLE migrations OWNER TO elibrary;
ALTER TABLE pictures OWNER TO elibrary;
```

## Dump db for backup
`pg_dump -U postgres elibrary_db > eLibrary.dump`
	
## Dump a TABLE
`pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "<path>/<tableName.sql>" --table <tableName> elibrary_db`

## Restoring table
Enter psql and run the command below.

`psql -U postgres`

### Connect to the eLibrary database
```shell
Postgres-# \c elibrary_db
Postgres-# \i tableName.sql
```
Drop the current DB
```
dropdb -U postgres elibrary_db
```

Open PSQL
```
psql -U postgres
```
Create new Scoop DB
```
CREATE DATABASE elibrary_db;
```

To exit PSQL type:
```
\q
```

Restore the database
```
psql -U postgres -d elibrary_db -f <BackupFile>.sql
```	
	

# eLibary manager location

`/opt/elibrary`

# Running eLibrary importer

`./opt/elibrary/manager.py import`