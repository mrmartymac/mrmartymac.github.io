---
title: How to configure logrotate
date: 2023-03-08 15:12:45 -500
categories: [cat1, cat2]
tags: [tag1, tag2] # TAG names should always be lowercase
author: mm
---
# The purpose of logrotate

This function will take care of compressing and removing logs from systems. 

# Installing logrotate

`yum install logrotate`

# Files to edit

There is a file for collections of logs to be managed.  The two primary logrotate files to edit are **httpd** and **syslog**

`cd /etc/logrotate.d`

Edit **httpd** and **syslog** the exmaple below are the additions made to httpd, but both files should be configured similarly with attention to **daily** and **maxage 7**

```bash
http
/var/log/httpd/*log {
missingok
notifempty
sharedscripts
delaycompress
daily
maxage 7
postrotate
/bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
endscript
}
```