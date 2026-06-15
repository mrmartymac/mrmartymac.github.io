---
title: CAS dreclog fix
date: 2026-04-27 08:24:05 -400
categories: [Cheatsheets, CAS]
tags: [cas, dreclog] # TAG names should always be lowercase
author: mm
---
Symptom: Going to any CAS page does not return a response, it eventually times out. The server load is almost always low when this happens. You may also have also have users in X11 applications (AdMAX, Layout) that have stuck processes. Usually you can confirm the issue by running 
```bash
/var/www/cgi-bin/scs/all/spice.pl
```
directly on the server. If everything is working as expected, you should get an HTML page dumped to your terminal. If it hangs it is mostly likely a full dreclog file, CTRL-C the process and move on.

Cause: `/u/scs/tools/data/dreclog.dat` or `/u/scs/tools/data/dreclog.idx` are 2GB. You can run du on these to check their size.

Fix: As root:
```bash
cd /u/scs/tools/data
/u/scs/tools/bin/makrecs dreclog ../tbin/dreclog
chmod 666 dreclog.dat dreclog.idx
```
It is also a good idea to cleanup the stuck processes, which can be identified by looking at anything that has open the old (_o) versions of the files: 
```bash
lsof /u/scs/tools/data/dreclog.dat_o
lsof /u/scs/tools/data/dreclog.idx_o
```

Any processes with those old files should be killed using kill -9 pid