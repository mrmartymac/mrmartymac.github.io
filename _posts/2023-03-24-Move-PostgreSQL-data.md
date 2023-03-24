---
title: How to move PostgreSQL data 
date: 2023-03-24 08:18:26 -500
categories: [Cheatsheet, PostgreSQL]
tags: [potgres, postgresql, pgsql, synlink] # TAG names should always be lowercase
author: mm
---
# How to move PostgreSQL data   

Moving the data folder for postgresql from the default location to another partition and creating a symlink.

> Be sure to do both steps on the Primary and DR server if one exists.
{: .prompt-warning }

```bash
sudo pg_dump -U postgres scoop_db > <date>_scoop_db.dump
sudo systemctl stop murbsd
sudo systemctl stop ScoopDaemon
sudo systemctl stop postgresql
```
>Elevate to root
{: .prompt-info }

```bash
mv /var/lib/pgsql/* /u/postgres/
sudo ln -s /u/postgres /var/lib/pgsql
```

```bash
sudo systemctl start postgresql
sudo systemctl start ScoopDaemon
sudo systemctl start murbsd
```