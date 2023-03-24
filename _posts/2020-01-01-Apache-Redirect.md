---
title: Configure Apache Redirect
date: 2023-02-20 15:32:13 -500
categories: [Cheatsheet, Apache]
tags: [apache, web] # TAG names should always be lowercase
author: mm
---
The only line that is important is the RedirectMatch ... line, however, it's location may be important so I included surrounding code...

Edit `/etc/httpd/conf/httpd.conf`

```
<IfModule alias_module>
    #
    # Redirect: Allows you to tell clients about documents that used to
    # exist in your server's namespace, but do not anymore. The client
    # will make a new request for the document at its new location.
    # Example:
    # Redirect permanent /foo http://www.example.com/bar
RedirectMatch ^/$ http://scoop.newspapersystems.com/WebEdit
```

If you want to be a bit more "professional", you should probably change "http" to "https", though haproxy should redirect if you do not. Also, of course, you can modify the /WebEdit to be whatever you would like; for our site, I decided WebEdit was the place to go to...
This is in  by the way, meant to include that at the end

Also you'll need to do a `systemctl restart httpd` for the changes to take effect