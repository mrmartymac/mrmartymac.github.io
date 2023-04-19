---
title: FotoWeb Migration to new server
date: 2023-04-18 14:02:39 -400
categories: [FotoWeb, Migration]
tags: [fotoware, fotoweb, migration] # TAG names should always be lowercase
author: mm
---
 # Overview
 For the best results, install the same verion of FotoWeb on the current instance on the new server and set up a placeholder site with the same site name, domain. Preferably use the same Process Account.
 
## Files to copy
Stop all the FotoWare services (on both servers) and copy:  

`C:\ProgramData\FotoWare\Metadata\MetadataConfiguration.xml`

`C:\ProgramData\FotoWare\FotoWeb\Site Settings\{sitename}`   `C:\ProgramData\FotoWare\FotoWeb\Operations\MongoDBData`  

Check that the copied site works then gradually upgrade FotoWeb by installing each Feature Release version and test as you go until you finish with the latest FotoWeb 8.1. Suggest backing up the Site Settings and MongoDBData folder before each upgrade so you can recover the settings and try again if there are any problems. This method does not disrupt the old site so the users can still access their old FotoWeb site while you work on the migration.