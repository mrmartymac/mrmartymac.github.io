---
title: 
date: 2023-11-07 12:01:15 -400
categories: [Cheatsheet, SSL Certificate]
tags: [ssl, certificate, httpd] # TAG names should always be lowercase
author: mm
---
## HAPROXY APPROACH USED BY ADVERTISING
### Go to the certs directory
```
cd /etc/pki/tls/certs/
```

### See that there is already a certificate PEM file - star.media.cherryroad.com.full.pem (this is a combo of the CERT, INTERMIDIATE FILE and KEY)
```
ls
```
`ca-bundle.crt        star.media.cherryroad.com.crt`  
`ca-bundle.trust.crt  star.media.cherryroad.com.full.pem`   
`sendmail.pem         star.media.cherryroad.com.intermediate.crt ` 

### Check for the KEY file, it should be up-one-level in the private directory
```
ls ../private/
```
`sendmail.key  star.media.cherryroad.com.key`

### Backup the existing certificate, just in-case.
```
cp star.media.cherryroad.com.full.pem star.media.cherryroad.com.full.pem.20231107
```

### CAT the new certificate, intermidiate file and the private key into the PEM file.
```
cat /root/star.media.cherryroad.com.crt star.media.cherryroad.com.intermediate.crt ../private/star.media.cherryroad.com.key > star.media.cherryroad.com.full.pem
```

### Restart haproxy (the web service)
```
systemctl restart haproxy.service
```

## HTTPD APPROACH USED BY SCOOP
### Go to the certs directory
```
cd /etc/pki/tls/certs/
```

### Backup the current certificate
```
cp star.media.cherryroad.com.crt star.media.cherryroad.com.crt.yyyymmdd
```

### Replace the original certificate with the new certificate
```
cp xxx.xxx.xxx.crt /etc/pki/tls/certs/xxx.xxx.xxx.crt
```

### Restart httpd
```
systemctl restart httpd
```