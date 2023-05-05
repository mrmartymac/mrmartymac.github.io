---
title: Scoop WordPress Parser
date: 2023-05-02 12:41:21 -500
categories: [cat1, cat2]
tags: [tag1, tag2] # TAG names should always be lowercase
author: mm
---

Queries for the IP address change

```sql
select * from wp_options where option_name = 'siteurl';
update wp_options set option_value='http://10.0.0.184' where option_name='siteurl';
update wp_options set option_value='http://10.0.0.184' where option_name='home';
```

Endpoints
```bash
http://10.0.0.184/wp-json/jwt-auth/v1/
http://10.0.0.184/wp-json/wp/v2/posts
```
Be sure to go to the settings page and save the settings again.

```curl
curl --location --request POST 'http://10.0.0.184/wp-json/jwt-auth/v1/token' \
--header 'Content-Type: application/json' \
--data '{
    "username": "mmacdonald",
    "password": "2010RedBank" 
}'
```

Plug-ins to add
authors slug
wp-mediabox-webparser-master
