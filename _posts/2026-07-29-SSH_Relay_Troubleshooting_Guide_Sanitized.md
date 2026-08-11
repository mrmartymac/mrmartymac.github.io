---
title: SSH Relay Troubleshooting Guide
date: 2026-07-29 14:11:51 -400
categories: [Documentation, SSH Relay]
tags: [ssh, relay, troubleshoot] # TAG names should always be lowercase
author: mm
---

> **Public-safe edition:** Infrastructure-specific values, fingerprints, account names, group names, and defensive thresholds have been replaced with placeholders. Obtain production values from an authorized administrator through a trusted channel.


This guide documents the most common problems that can prevent users from connecting to `<RelayHostAlias>`, along with the checks and corrective actions used during deployment.

## Relay Connection Details

```text
Public IP: <PublicIPAddress>
Public SSH Port: <ExternalSSHPort>
Internal Relay IP: <InternalRelayIPAddress>
Internal SSH Port: 22
SSH Alias: <RelayHostAlias>
Required Authentication: SSH public key + TOTP
Allowed Group: <AuthorizedSSHGroup>
```

Expected firewall translation:

```text
<PublicIPAddress>:<ExternalSSHPort>
        ↓
<InternalRelayIPAddress>:22
```

---

# 1. Start With the Symptom

## Timeout Before Authentication

Example:

```text
Connecting to <PublicIPAddress> port <ExternalSSHPort>
Operation timed out
```

This means the client never reached SSH authentication.

Likely causes:

- Fail2ban banned the user's public IP
- Untangle port-forward rule is not matching
- Port-forward rule order is wrong
- Wrong destination IP or translated port
- Relay is unreachable
- Firewalld is blocking SSH
- Testing from inside the office without NAT loopback support
- Upstream network or ISP filtering

## Key Rejected

Example:

```text
The server refused your key
```

or verbose SSH shows `Offering public key` without `Server accepts key`.

Likely causes:

- Wrong public key in `authorized_keys`
- Wrong private key selected by client
- Bad permissions or ownership
- User not in `<AuthorizedSSHGroup>`
- SELinux label problem
- Corrupted or wrapped key line
- Client offered too many other keys first

## TOTP Prompt Appears, Then Connection Closes

Example:

```text
Verification code:
Connection closed
```

Likely causes:

- Wrong TOTP code
- TOTP secret mismatch
- Server or phone time drift
- Missing `.google_authenticator` file
- Wrong permissions or ownership
- SELinux denial
- `LoginGraceTime` exceeded

## Password Prompt Appears

The new relay is configured for:

```text
publickey,keyboard-interactive:pam
```

A password prompt often means:

- The client reached the wrong SSH server
- Untangle forwarded to the old relay
- The key was rejected and the client fell back
- The client application does not support the required authentication sequence

---

# 2. Watch SSH Login Attempts Live

```bash
sudo journalctl -u sshd -f
```

Have the user retry while this command is running.

If no new log lines appear, the connection is not reaching the relay.

Recent SSH events:

```bash
sudo journalctl -u sshd --since "30 minutes ago" --no-pager -l
```

Authentication-focused view:

```bash
sudo journalctl -u sshd --since "today" --no-pager |
grep -Ei 'accepted|failed|invalid user|authentication failure|disconnect|maximum authentication attempts|LoginGraceTime'
```

---

# 3. Fail2ban Checks

## Show Jail Status

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

Look for:

```text
Currently banned
Total banned
Banned IP list
```

## Determine the User's Public IP

From the user's Mac or Windows PC:

```bash
curl https://api.ipify.org
echo
```

## Unban an IP

```bash
sudo fail2ban-client set sshd unbanip USER.PUBLIC.IP.ADDRESS
```

## Show Active Thresholds

```bash
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd findtime
sudo fail2ban-client get sshd bantime
```

Original configuration:

```ini
maxretry = 5
findtime = 10m
bantime = 1h
```

Five matching failures within ten minutes results in a one-hour ban.

To raise the threshold during testing:

```bash
sudo nano /etc/fail2ban/jail.d/sshd.local
```

Example with placeholders:

```ini
[sshd]
enabled = true
backend = systemd
banaction = firewallcmd-rich-rules
port = ssh
maxretry = <MaximumRetries>
findtime = <FailureWindow>
bantime = <BanDuration>
```

Validate and restart:

```bash
sudo fail2ban-client -t
sudo systemctl restart fail2ban
```

Verify:

```bash
sudo fail2ban-client get sshd maxretry
```

## Check Ignored Networks

```bash
sudo fail2ban-client get sshd ignoreip
```

Expected internal exceptions:

```text
127.0.0.0/8
<InternalNetworkCIDR>
::1
```

## Service Checks

```bash
sudo systemctl status fail2ban --no-pager -l
sudo journalctl -u fail2ban --since "15 minutes ago" --no-pager -l
sudo fail2ban-client ping
```

Expected:

```text
Server replied: pong
```

If the socket is briefly unavailable immediately after restart, wait a few seconds and retry.

---

# 4. Confirm the User Exists and Is Allowed

```bash
getent passwd <YourUserName>
id <YourUserName>
```

The user must belong to `<AuthorizedSSHGroup>` and normally should not belong to `wheel`.

Because the relay uses:

```text
AllowGroups <AuthorizedSSHGroup>
```

a valid Linux account outside that group will be denied.

Add the user to the allowed group:

```bash
sudo usermod -a -G <AuthorizedSSHGroup> <YourUserName>
```

Check account expiration and password status:

```bash
sudo chage -l <YourUserName>
sudo passwd -S <YourUserName>
```

A locked password is acceptable because SSH uses public keys and TOTP.

---

# 5. Check the Home Directory and Shell

```bash
getent passwd <YourUserName>
```

Expected format:

```text
<YourUserName>:x:UID:GID:Name:/home/<YourUserName>:/bin/bash
```

Check the home directory:

```bash
sudo ls -ld /home/<YourUserName>
```

A missing home directory or shell such as `/sbin/nologin` can prevent a normal session.

---

# 6. Verify the Public Key

## On Mac

```bash
ssh-keygen -lf ~/.ssh/ssh-relay-<YourUserName>.pub
```

## On Windows PowerShell

```powershell
ssh-keygen -lf "$HOME\.ssh\ssh-relay-<YourUserName>.pub"
```

## On the Relay

```bash
sudo ssh-keygen -lf /home/<YourUserName>/.ssh/authorized_keys
```

The fingerprints must match exactly.

Inspect the file:

```bash
sudo cat -A /home/<YourUserName>/.ssh/authorized_keys
```

Look for wrapped lines, extra characters, pasted prompts, `^M`, or the wrong user's key. Each key must be one complete line.

---

# 7. Repair Ownership and Permissions

```bash
sudo chown -R <YourUserName>:<YourUserName> /home/<YourUserName>/.ssh
sudo chmod 700 /home/<YourUserName>
sudo chmod 700 /home/<YourUserName>/.ssh
sudo chmod 600 /home/<YourUserName>/.ssh/authorized_keys
sudo chmod 600 /home/<YourUserName>/.ssh/.google_authenticator
sudo restorecon -RFv /home/<YourUserName>/.ssh
```

Verify:

```bash
sudo stat -c '%U:%G %a %n' \
  /home/<YourUserName> \
  /home/<YourUserName>/.ssh \
  /home/<YourUserName>/.ssh/authorized_keys \
  /home/<YourUserName>/.ssh/.google_authenticator
```

Typical values:

```text
<YourUserName>:<YourUserName> 700 /home/<YourUserName>
<YourUserName>:<YourUserName> 700 /home/<YourUserName>/.ssh
<YourUserName>:<YourUserName> 600 /home/<YourUserName>/.ssh/authorized_keys
<YourUserName>:<YourUserName> 600 /home/<YourUserName>/.ssh/.google_authenticator
```

---

# 8. Check SELinux

```bash
sudo ls -ldZ /home/<YourUserName>/.ssh
sudo ls -lZ /home/<YourUserName>/.ssh/authorized_keys
sudo ls -lZ /home/<YourUserName>/.ssh/.google_authenticator
```

The SSH files should normally use `ssh_home_t`.

Repair labels:

```bash
sudo restorecon -RFv /home/<YourUserName>/.ssh
```

Check recent denials:

```bash
sudo ausearch -m AVC,USER_AVC -ts recent
```

---

# 9. Check TOTP Enrollment

Verify the file exists:

```bash
sudo ls -l /home/<YourUserName>/.ssh/.google_authenticator
```

Check ownership and permissions:

```bash
sudo stat -c '%U:%G %a %n' /home/<YourUserName>/.ssh/.google_authenticator
```

Confirm PAM points to the expected file:

```bash
sudo grep pam_google_authenticator /etc/pam.d/sshd
```

Expected:

```text
auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator
```

Check server time:

```bash
timedatectl
chronyc tracking
```

A phone with incorrect automatic time settings can generate invalid TOTP codes.

---

# 10. Check Login Grace Time

The original setting was too short:

```text
LoginGraceTime 30
```

The working value is:

```text
LoginGraceTime 120
```

Check it:

```bash
sudo sshd -T | grep -i logingracetime
```

Confirm that the effective value matches your organization’s approved setting.

If needed, edit:

```bash
sudo nano /etc/ssh/sshd_config.d/00-ssh-relay.conf
```

Then validate and reload:

```bash
sudo sshd -t
sudo systemctl reload sshd
```

A log entry like this confirms the problem:

```text
penalty: exceeded LoginGraceTime
```

---

# 11. Verify Effective SSH Configuration

```bash
sudo sshd -T |
grep -Ei '^(allowgroups|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|authenticationmethods|maxauthtries|logingracetime|allowtcpforwarding|x11forwarding) '
```

Expected values include:

```text
allowgroups <AuthorizedSSHGroup>
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication yes
authenticationmethods publickey,keyboard-interactive:pam
maxauthtries 3
logingracetime 120
allowtcpforwarding yes
x11forwarding yes
```

Validate syntax:

```bash
sudo sshd -t
```

---

# 12. Mac Client Checks

## Full Direct Command

```bash
ssh \
  -o IdentitiesOnly=yes \
  -i ~/.ssh/ssh-relay-<YourUserName> \
  -p <ExternalSSHPort> \
  <YourUserName>@<PublicIPAddress>
```

## Recommended Config Entry

```sshconfig
Host <RelayHostAlias>
    HostName <PublicIPAddress>
    Port <ExternalSSHPort>
    User <YourUserName>
    IdentityFile ~/.ssh/ssh-relay-<YourUserName>
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes
```

Verify applied settings:

```bash
ssh -G <RelayHostAlias> |
grep -Ei '^(hostname|user|port|identityfile|identitiesonly) '
```

## Clear a Wrong Keychain Passphrase

```bash
ssh-add -d ~/.ssh/ssh-relay-<YourUserName>
```

Or clear all loaded keys:

```bash
ssh-add -D
```

Delete the matching OpenSSH passphrase entry in Keychain Access, then re-add:

```bash
ssh-add --apple-use-keychain ~/.ssh/ssh-relay-<YourUserName>
```

Check loaded keys:

```bash
ssh-add -l
```

Verbose test:

```bash
ssh -vvv -o ConnectTimeout=10 <RelayHostAlias>
```

---

# 13. Windows Client Checks

## Full Direct Command

```powershell
ssh `
  -o IdentitiesOnly=yes `
  -i "$HOME\.ssh\ssh-relay-<YourUserName>" `
  -p <ExternalSSHPort> `
  <YourUserName>@<PublicIPAddress>
```

## Recommended Config Entry

```sshconfig
Host <RelayHostAlias>
    HostName <PublicIPAddress>
    Port <ExternalSSHPort>
    User <YourUserName>
    IdentityFile ~/.ssh/ssh-relay-<YourUserName>
    IdentitiesOnly yes
```

Verify applied settings:

```powershell
ssh -G <RelayHostAlias> |
    Select-String '^(hostname|user|port|identityfile|identitiesonly) '
```

Enable Windows `ssh-agent` from an elevated PowerShell window:

```powershell
Get-Service ssh-agent |
    Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Then, as the user:

```powershell
ssh-add "$HOME\.ssh\ssh-relay-<YourUserName>"
ssh-add -l
```

---

# 14. Too Many Authentication Failures

Example:

```text
Too many authentication failures
```

This usually means the client offered multiple keys before the correct one.

Use:

```text
IdentitiesOnly yes
```

and specify the correct key explicitly.

Remember that `MaxAuthTries` is low, so several offered keys can consume the allowance quickly.

---

# 15. Changed Host-Key Warning

Example:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Verify the relay fingerprint:

```bash
sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Known fingerprint during deployment:

```text
<ApprovedED25519Fingerprint>
```

After verifying, remove the old entry:

```bash
ssh-keygen -R "[<PublicIPAddress>]:<ExternalSSHPort>"
```

Reconnect and accept only if the fingerprint matches.

---

# 16. Untangle Port-Forwarding Checks

The rule must map:

```text
TCP <PublicIPAddress>:<ExternalSSHPort>
        ↓
<InternalRelayIPAddress>:22
```

Confirm:

- Rule is enabled
- Protocol is TCP
- External destination port is `<ExternalSSHPort>`
- Internal destination IP is `<InternalRelayIPAddress>`
- Internal destination port is `22`
- Rule is attached to the correct WAN interface
- Rule order is above broader conflicting rules
- No unexpected source-IP restriction is present

Rule ordering caused an actual deployment issue. An earlier matching rule can capture traffic before the new relay rule is evaluated.

---

# 17. Test Port Reachability

From Mac:

```bash
nc -vz -w 10 <PublicIPAddress> <ExternalSSHPort>
```

Interpretation:

- `succeeded`: port is reachable
- `Operation timed out`: traffic is dropped or misrouted
- `Connection refused`: destination was reached, but nothing accepted the connection

On the relay:

```bash
sudo ss -lntp | grep sshd
```

Expected:

```text
0.0.0.0:22
[::]:22
```

Check firewalld:

```bash
sudo firewall-cmd --list-all
```

You should see the `ssh` service or TCP port 22 allowed.

---

# 18. NAT Loopback Consideration

If a user tests from inside the office using the public address, the connection may fail if the firewall does not support NAT loopback.

Direct internal test:

```bash
ssh \
  -o IdentitiesOnly=yes \
  -i ~/.ssh/ssh-relay-<YourUserName> \
  <YourUserName>@<InternalRelayIPAddress>
```

Optional internal alias:

```sshconfig
Host <RelayHostAlias>-local
    HostName <InternalRelayIPAddress>
    Port 22
    User <YourUserName>
    IdentityFile ~/.ssh/ssh-relay-<YourUserName>
    IdentitiesOnly yes
```

---

# 19. WebSSH Compatibility Notes

WebSSH must authenticate in this order:

```text
public key
then
keyboard-interactive TOTP
```

Do not force keyboard-interactive before key authentication.

Recommended settings:

```text
Host: <PublicIPAddress>
Port: <ExternalSSHPort>
Username: <YourUserName>
Authentication: Private Key
Private Key: user's imported private key
Force Keyboard Interactive: Off
Password: blank
```

A log showing `authList>publickey</authList>` means the server is still waiting for the public-key step.

---

# 20. Confirm Active Users

```bash
who
w
who -u
```

Unique usernames:

```bash
who | awk '{print $1}' | sort -u
```

SSH processes:

```bash
ps -ef | grep '[s]shd: .*@'
```

---

# 21. Check Idle Timeout Settings

```bash
sudo sshd -T |
grep -Ei '^(clientaliveinterval|clientalivecountmax|logingracetime) '
```

Current values:

```text
logingracetime 120
clientaliveinterval 0
clientalivecountmax 3
```

`ClientAliveInterval 0` means SSH itself does not enforce an idle-session timeout.

Check shell-level timeouts:

```bash
sudo grep -Rni '^[[:space:]]*TMOUT=' \
  /etc/profile \
  /etc/bashrc \
  /etc/profile.d 2>/dev/null
```

---

# 22. System Health Checks

```bash
sudo systemctl status sshd --no-pager -l
sudo systemctl status fail2ban --no-pager -l
df -h
df -i
timedatectl
chronyc tracking
```

A full filesystem, inode exhaustion, time drift, or failed service can prevent successful logins.

---

# 23. Recommended Troubleshooting Sequence

1. Ask for the user's current public IP.
2. Check Fail2ban.
3. Run `journalctl -u sshd -f`.
4. Have the user retry.
5. If no logs appear, check Untangle and reachability.
6. If logs appear, read the exact rejection reason.
7. Verify membership in `<AuthorizedSSHGroup>`.
8. Compare SSH key fingerprints.
9. Repair ownership, permissions, and SELinux labels.
10. Verify the TOTP file and system time.
11. Confirm effective `sshd -T` settings.
12. Use verbose client logging.

---

# 24. Compact Administrator Command Set

```bash
# Live SSH logs
sudo journalctl -u sshd -f

# Fail2ban
sudo fail2ban-client status sshd

# User and group
id <YourUserName>

# Key fingerprint
sudo ssh-keygen -lf /home/<YourUserName>/.ssh/authorized_keys

# Permissions
sudo stat -c '%U:%G %a %n' \
  /home/<YourUserName> \
  /home/<YourUserName>/.ssh \
  /home/<YourUserName>/.ssh/authorized_keys \
  /home/<YourUserName>/.ssh/.google_authenticator

# SELinux
sudo restorecon -RFv /home/<YourUserName>/.ssh

# Effective SSH settings
sudo sshd -T |
grep -Ei '^(allowgroups|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|authenticationmethods|maxauthtries|logingracetime) '

# Validate SSH
sudo sshd -t
```

---

# 25. XWindows or X11 fails

If XWindows fail to launch or `echo $DISPLAY` returns a blank line. Please run the command below.

```bash
echo "$DISPLAY"
ls -l ~/.Xauthority
xauth list
```

---

# 26. Information to Collect From a User

Ask the user to provide:

- Assigned relay username
- Current public IP
- Mac or Windows
- Exact SSH command used
- Exact error message
- Output from `ssh -vvv`
- Public-key fingerprint
- Approximate time of the failed attempt

Do not ask them to send:

- Private key
- Private-key passphrase
- TOTP secret
- Current TOTP code
- Emergency recovery codes
