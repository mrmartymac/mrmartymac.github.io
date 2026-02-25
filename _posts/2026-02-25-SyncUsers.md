---
title: Fix User Sync in mirroring
date: 2026-02-25 10:05:18 -400
categories: [Cheatsheets, Mirror]
tags: [mirroring, users] # TAG names should always be lowercase
author: mm
---
This will address the issue of user accounts syncing across and replacing unique passwords.

```bash
ncftpget -u ftpaccess -p knuckle ftp.newspapersystems.com . ethan/sync_users.sh
chmod 755 sync_users.sh
chown root:root sync_users.sh
yes | cp -i sync_users.sh /u/scs/tools/bin/sync_users.sh
yes | cp -i sync_users.sh /u/scs/tools/compiler/sync_users.sh
rm -f sync_users.sh
ll /u/scs/tools/bin/sync_users.sh
ll /u/scs/tools/compiler/sync_users.sh
echo "Done"

```

### Original Note
1. Get sync_users.sh script from Ethan FTP
2. Place in the following directories:
  /u/scs/tools/compiler/sync_users.sh
  /u/scs/tools/bin/sync_users.sh