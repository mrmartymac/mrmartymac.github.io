---
title: eLibrary information
date: 2023-03-09 14:28:40 -500
categories: [Documentation, eLibrary]
tags: [elibrary] # TAG names should always be lowercase
author: mm
---
# Installation

Download the rpm to the server and extract it 
```bash
tar xvzf eLibrary-<version>.tar.gz
```
Run the installation script
```bash
cd eLibrary-<version>
./install_elibrary.sh
```

Configuration file location
```bash
/etc/xdg/SCS/eLibrary.Importer.ini
```

# Manually running importer
```bash
cd /opt/elibrary
./manager.py import
```

# Automated importer process
During the installation a cron job is created in `/etc/cron.d` called **elibrary-maintainer**. The default timing for the importer to run is 03:00 followed at 04:00 with a job that cleans up the backup directory.  At the time of this writing the default configuration is as shown below.
```bash
# Scheduled eLibrary maintenance tasks in cron.
SHELL=/bin/bash
PATH=/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin
MAILTO=scoop

# Run the archiver at 3:00 am nightly
0 3 * * * scoop /opt/elibrary/manager.py import >>/var/log/elibrary/importer.log 2>&1

# Cleanup old XML files at 4:00 am nightly
0 4 * * * scoop /opt/elibrary/manager.py cleanup_backup >>/var/log/elibrary/cleanup_backup.log 2>&1
```
## Definition of cleanup_backup  
The time frame for the cleanup is specified in the following file with the **XMLCleanupDays** parameter
```bash
/etc/xdg/SCS/eLibrary.Importer.ini
```

# Database configuration

## Drop the current DB
```
dropdb -U postgres elibrary_db
```

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
cat <installDir>/elibrary.sql | sudo -i -u postgres psql -U elibrary elibrary_db
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
```sql
pg_dump -U postgres elibrary_db > eLibrary.dump
```
## Restore the database
```sql
psql -U postgres -d elibrary_db -f <BackupFile>.sql
```	


# Working with tables  	
## Dump a TABLE
```bash
pg_dump --host localhost --port 5432 --username postgres --format plain --verbose --file "<path>/<tableName.sql>" --table <tableName> elibrary_db
```

## Restoring table
Enter psql and run the command below.
```sql
psql -U postgres
\c elibrary_db
\i tableName.sql
```

## Addendum
Upgrade pip because why not
```
python3 -m pip install --upgrade pip
```

Install pypdfium2
```
python3 -m pip install -U pypdfium2
```
	
