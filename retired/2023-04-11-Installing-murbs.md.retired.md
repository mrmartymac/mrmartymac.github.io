---
title: Installing murbs
date: 2023-04-11 08:26:40 -400
categories: [Installation, MURBS]
tags: [murbs, backup, install] # TAG names should always be lowercase
author: mm
---

yum install httpd  
yum install ftp  
Get scs-moai-email-1.0.0-2.aarch64.rpm from FTP under marty  
sudo rpm -i scs-moai-email-1.0.0-2.aarch64.rpm  

```
| |
 _ __ ___  __ _ __| |__  ___
| '_ ` _ \| | | | '__| '_ \/ __|
| | | | | | |_| | |  | |_) \__ \
|_| |_| |_|\__,_|_|  |_.__/|___/
```

*The Manual*

Installation
--------------------------------
This process should only be performed once, on the (initial) primary.

1. `yum install scs-murbs...rpm`
2. Run `murbs init` (as root.)
- This will create the `murbs` user, set up configuration directories, etc.
** MM: running this the first time resulted in errors, running it a second time resulted in being prompted for the murbs password which was not yet set.  I ran passwd to set a password for murbs user.  Then the command completed after confirming the ip address of the machine and using the password.**

3. Allow the `wheel` group to run `sudo` with no password.
```
usermod -aG wheel murbs
```

- This is necessary for `murbsd` to be able to sudo.
- Just run `visudo`, locate the `## Same thing without a password` line, and uncomment it.
- Type `:x` to save and exit

After installation, you will want to enable and configure modules; see the following section on that.


Making a Machine the Primary
--------------------------------
1. Run `murbs primary` on the machine you want to make primary.
- This may ask you to confirm what you are doing.
1. That's it. (You may want to check the log to confirm everything started normally.)


Adding a Machine for Replication
---------------------------------
1. Synchronize users, home directories, and the sudoers file with the secondary (i.e. using old SCS Mirroring, or manually.)
2. Install `scs-murbs...rpm` on the secondary.
3. Run (on primary) `murbs machine add <ipaddress>`.
4. Restart `murbsd`.
5. Check the logs (`journalctl -u murbsd`) to make sure everything went well.

Note that if you already enabled some modules, you will need to run `murbs fullsync`.


Enabling Modules
--------------------------------

### `postgresql`

1. If you want PostgreSQL's data directory to be somewhere other than the default, you must move it (on **each machine**) first.
- Ensure PostgreSQL is enabled first:
```
systemctl enable postgresql
```
- This is typically done using symlinks:
```
service postgresql stop
mv /var/lib/pgsql/ /u/postgres
ln -s /u/postgres/ /var/lib/pgsql
service postgresql start
```
2. Configure `systemd` **on each machine** to not wait for `postgres` to start up.
- Otherwise, the 5-minute timeout for non-startup will be hit on the secondary(ies), since these just block waiting for the incoming WAL files.
- This is done by running `systemctl edit postgresql`, and then inputting the following:
```
[Service]
Type=forking
ExecStart=
ExecStart=/usr/bin/pg_ctl start -D "${PGDATA}" -s -W
```
Note that the first, empty `ExecStart=` is **required.**
- If old SCS mirroring is active, you must disable the `pgsql` entries in `mirror.include`.
3. Configure (on primary) the `postgresql` module: `murbs module configure postgresql`.
- This will edit `postgres.conf` and change all necessary settings, and then restart the PostgreSQL server.
4. Restart `murbsd`.
5. Check the logs (`journalctl -u murbsd`), and run a `fullsync` to push all changes out to the other machines.

# June 13 2024 - CRM Hosted Papers Install

1. `yum install scs-murbs...rpm`
2. Run `murbs init` (as root.)
- This will create the `murbs` user, set up configuration directories, etc.
** MM: running this the first time resulted in errors, running it a second time resulted in being prompted for the murbs password which was not yet set.  I ran passwd to set a password for murbs user.  Then the command completed after confirming the ip address of the machine and using the password.**
3. Set the password for mursb
```
passwd murbs
```
4. Run the command again, entering the password when prompted
5. Had to runt he followong to get the ssh keys updated.
```
cat /home/murbs/.ssh/id_rsa.pub >> /home/murbs/.ssh/authorized_keys
```
6. Allow the `wheel` group to run `sudo` with no password.
```
usermod -aG wheel murbs
```
- This is necessary for `murbsd` to be able to sudo.
- Just run `visudo`, locate the `## Same thing without a password` line, and uncomment it.
- Type `:x` to save and exit or `esc :qx`
- 
2. Configure `systemd` **on each machine** to not wait for `postgres` to start up.
   - Otherwise, the 5-minute timeout for non-startup will be hit on the secondary(ies), since these just block waiting for the incoming WAL files.
   - This is done by running `systemctl edit postgresql`, and then inputting the following:
   ```
   [Service]
   Type=forking
   ExecStart=
   ExecStart=/usr/bin/pg_ctl start -D "${PGDATA}" -s -W
   ```
   Note that the first, empty `ExecStart=` is **required.**
   - If old SCS mirroring is active, you must disable the `pgsql` entries in `mirror.include`.
3. Configure (on primary) the `postgresql` module: 
   ```
   murbs module configure postgresql
   ```
   - This will edit `postgres.conf` and change all necessary settings, and then restart the PostgreSQL server.
4. Restart murbsd
   ```
   systemctl restart murbsd
   ```
5. On the Primary
    ```
    touch /etc/murbs/primary    
    systemctl enable murbsd.service
    systemctl start murbsd.service
    systemctl status murbsd.service
    ```

    On secondary sync users
    useradd murbs -g daemon -m -G wheel
    passwd murbs
    visudo (uncomment same line on backup)

On Primary
murbs machine add <IP ADDERSS>
systemctl restart murbsd
systemctl status murbsd
murbs fullsync

6. Check the logs (`journalctl -u murbsd`), and run a `fullsync` to push all changes out to the other machines.