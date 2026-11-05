---
title: Sending email from command line
date: 2026-10-06 08:41:53 -400
categories: [Cheatsheets, email]
tags: [email, smtp, commandline] # TAG names should always be lowercase
author: mm
---

[scs@scoop ~]$ telnet smtpmail.lnpnews.com 25
Trying 192.168.22.4...
Connected to smtpmail.lnpnews.com.
Escape character is '^]'.

220 EX1.lnpnews.com Microsoft ESMTP MAIL Service ready at Wed, 9 Jun 2021 10:42:30 -0400
HELO
250 EX1.lnpnews.com Hello [192.168.22.86]
MAIL FROM: <scoop@lnpnews.com>
250 2.1.0 Sender OK
RCPT TO: <marty@newspapersystems.com>
250 2.1.5 Recipient OK
DATA 
354 Start mail input; end with <CRLF>.<CRLF>
From: scoop@lnpnews.com
To: marty@newspapersystems.com
Subject: Testing


Sample body of email
^]  <-- This is the pressing of CTRL+] to escape to the command line
telnet> set crlf
Will send carriage returns as telnet <CR><LF>.



. <-- type period and press enter
250 2.6.0 <75c5dc27-30ec-4204-ab8b-fd3c412e42f7@EX1.lnpnews.com> [InternalId=157346126889145, Hostname=EX1.lnpnews.com] 1505 bytes in 2:33.217, 0.010 KB/sec Queued mail for delivery

Independent 