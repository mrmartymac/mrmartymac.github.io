---
title: Postgres Commands
date: 2023-03-01 13:47:01 -500
categories: [Cheatsheet, PostgreSQL]
tags: [postgres, psql] # TAG names should always be lowercase
author: mm
---
# Using Postgres from the CLI

## To enter Postgres type

```bash
psql -U postgres
```

To list databases  
```bash
\l
```

Connect to a database  
```bash
\c <dbName>
```

List tables  
```bash
\dt
```

List tables and size  
```bash
\dt+
```

List columns  
```bash
\d+ <table-name>
```

## Export query results to CSV

**Users**
```sql
COPY (SELECT name, real_name, email, "group", permissions FROM public.users) TO '/u/pagetrack/SCOOPInstallationDirectory/users.csv' (format csv, delimiter ';');
```
**Products**
```sql
COPY (SELECT id, name FROM public.products) TO '/u/pagetrack/SCOOPInstallationDirectory/products.csv' (format csv, delimiter ';');
```

Restore a table

```sql
pg_restore -a -t your_table /path/to/dump.sql
```

Dump a table
articles
article_texts
article_pdfs
pictures
fullpage_pictures
doc_pdfs
migrations

```sql
pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\articles.sql" --table articles elibrary_db
pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\article_texts.sql" --table article_texts elibrary_db
pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\article_pdfs.sql" --table article_pdfs elibrary_db
pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\pictures.sql" --table pictures elibrary_db
pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\fullpage_pictures.sql" --table fullpage_pictures elibrary_db
pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\doc_pdfs.sql" --table doc_pdfs elibrary_db
pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\migrations.sql" --table migrations elibrary_db
```


## eLibrary Dump Database Script
```shell
@echo Started Articles: %date% %time% >> eLibraryDump.log
D:\SCOOP\PostgreSQL\9.6\bin\pg_dump.exe --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\articles.sql" --table articles elibrary_db >> eLibraryDump.log

@echo Started Article_texts: %date% %time% >> eLibraryDump.log
D:\SCOOP\PostgreSQL\9.6\bin\pg_dump.exe --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\article_texts.sql" --table article_texts elibrary_db >> eLibraryDump.log

@echo Started Aricle_pdfs: %date% %time% >> eLibraryDump.log
D:\SCOOP\PostgreSQL\9.6\bin\pg_dump.exe --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\article_pdfs.sql" --table article_pdfs elibrary_db >> eLibraryDump.log

@echo Started Pictures: %date% %time% >> eLibraryDump.log
D:\SCOOP\PostgreSQL\9.6\bin\pg_dump.exe --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\pictures.sql" --table pictures elibrary_db >> eLibraryDump.log

@echo Started Fullpage_Pictures: %date% %time% >> eLibraryDump.log
D:\SCOOP\PostgreSQL\9.6\bin\pg_dump.exe --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\fullpage_pictures.sql" --table fullpage_pictures elibrary_db >> eLibraryDump.log

@echo Started Doc_pdfs: %date% %time% >> eLibraryDump.log
D:\SCOOP\PostgreSQL\9.6\bin\pg_dump.exe --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\doc_pdfs.sql" --table doc_pdfs elibrary_db >> eLibraryDump.log

@echo Started Migrations: %date% %time% >> eLibraryDump.log
D:\SCOOP\PostgreSQL\9.6\bin\pg_dump.exe --host localhost --port 5432 --username postgres --format plain --verbose --file "\\scs3\PageTrack\ScoopInstallation\Server\eLibrary\migrations.sql" --table migrations elibrary_db >> eLibraryDump.log
```
