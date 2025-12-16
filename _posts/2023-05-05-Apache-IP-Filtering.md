---
title: IP Address Filtering for Apache
date: 2023-05-05 13:10:03 -400
categories: [Cheatsheets, Apache]
tags: [apache, httpd, filter, whitelist, virtualhost] # TAG names should always be lowercase
author: mm
---

There is some variability in this implementation of this.  Ideally it is entered in `/etc/httpd/conf/httpd.conf` but it is possible depending on the configuration.

On scoop.newspapersystems.com the VirtualHost for port 443 was contained in `/etc/httpd/conf/httpd-le-ssl.conf` in the `<Virtualhost>` node.

```bash
<VirtualHost *:443>
	<Location />
		Order deny,allow
		Deny from all
		Allow from 208.58.24.118
	</Location>
</VirtualHost>
```

Adding multiple addresses can be done by adding lines of `Allow` for each address or by using the subnet argument.