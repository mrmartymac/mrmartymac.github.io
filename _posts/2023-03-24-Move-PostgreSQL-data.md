---
title: How to move PostgreSQL data 
date: 2023-03-24 08:18:26 -500
categories: [Cheatsheets, PostgreSQL]
tags: [potgres, postgresql, pgsql, synlink] # TAG names should always be lowercase
author: mm
---
Moving the data folder for postgresql from the default location to another partition and creating a symlink.

> Be sure to do both steps on the Primary and DR server if one exists.
{: .prompt-warning }

>Elevate to root
{: .prompt-info }

```bash
pg_dump -U postgres scoop_db > <date>_scoop_db.dump
systemctl stop murbsd
systemctl stop ScoopDaemon
systemctl stop postgresql
```

```bash
mkdir /u/postgres
chmod 700 /u/postgres
chown postgres:postgres /u/postgres
mv /var/lib/pgsql/ /u/postgres/
ln -s /u/postgres /var/lib/pgsql
```

```bash
systemctl start postgresql
systemctl start ScoopDaemon
systemctl start murbsd
```