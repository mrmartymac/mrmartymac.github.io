---
title: SSH Relay User Onboarding Guide
date: 2026-07-28 08:39:18 -400
categories: [Documentation, SSH-Relays]
tags: [proxmox, ssh, relay] # TAG names should always be lowercase
author: mm
---


This guide documents the process for creating a new employee account on the AlmaLinux SSH relay and configuring that employee's Mac for public-key and TOTP authentication.

## Overview

Each employee should have:

- An individual Linux account on the relay
- Membership in the `ssh-access` group
- An individual SSH key pair
- An individual TOTP enrollment
- No shared account or shared private key
- No administrative access unless specifically required

Replace the example username `<YourUserName>` with the employee's actual Linux username.

---

# Part 1: Administrator Steps on the SSH Relay

## Step 1: Create the User Account

Log in to the SSH relay as an administrator and run:

```bash
sudo useradd \
  --create-home \
  --shell /bin/bash \
  --groups ssh-access \
  --comment "SSH relay user" \
  <YourUserName>
```

Verify the account:

```bash
getent passwd <YourUserName>
id <YourUserName>
sudo ls -ld /home/<YourUserName>
```

Confirm that:

- The home directory is `/home/<YourUserName>`
- The shell is `/bin/bash`
- The user belongs to `ssh-access`
- The user does not belong to `wheel`

Lock password authentication for the account:

```bash
sudo passwd -l <YourUserName>
```

Verify the password status:

```bash
sudo passwd -S <YourUserName>
```

A locked password does not prevent SSH public-key login.

---

## Step 2: Obtain the User's SSH Public Key

The employee must generate an SSH key on their Mac and send only the contents of the `.pub` file to the administrator.

The public key will look similar to:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... employee@Mac
```

Never request or accept the private key.

---

## Step 3: Create the User's `.ssh` Directory

Run:

```bash
sudo install \
  -d \
  -m 700 \
  -o <YourUserName> \
  -g <YourUserName> \
  /home/<YourUserName>/.ssh
```

Create the `authorized_keys` file:

```bash
sudo install \
  -m 600 \
  -o <YourUserName> \
  -g <YourUserName> \
  /dev/null \
  /home/<YourUserName>/.ssh/authorized_keys
```

---

## Step 4: Add the Public Key to `authorized_keys`

Open the file:

```bash
sudo nano /home/<YourUserName>/.ssh/authorized_keys
```

Paste the employee's complete public key on a single line.

Save and exit Nano:

1. Press `Ctrl+O`
2. Press `Enter`
3. Press `Ctrl+X`

Correct ownership, permissions, and SELinux labels:

```bash
sudo chown -R <YourUserName>:<YourUserName> /home/<YourUserName>/.ssh
sudo chmod 700 /home/<YourUserName>
sudo chmod 700 /home/<YourUserName>/.ssh
sudo chmod 600 /home/<YourUserName>/.ssh/authorized_keys
sudo restorecon -RFv /home/<YourUserName>/.ssh
```

Verify:

```bash
sudo stat -c '%U:%G %a %n' \
  /home/<YourUserName> \
  /home/<YourUserName>/.ssh \
  /home/<YourUserName>/.ssh/authorized_keys
```

Expected permissions:

```text
<YourUserName>:<YourUserName> 700 /home/<YourUserName>
<YourUserName>:<YourUserName> 700 /home/<YourUserName>/.ssh
<YourUserName>:<YourUserName> 600 /home/<YourUserName>/.ssh/authorized_keys
```

Check SELinux labels:

```bash
sudo ls -ldZ /home/<YourUserName>/.ssh
sudo ls -lZ /home/<YourUserName>/.ssh/authorized_keys
```

The `.ssh` content should normally use the `ssh_home_t` SELinux type.

---

## Step 5: Enroll TOTP for the User

Switch to the new user's account:

```bash
sudo -iu <YourUserName>
```

Verify:

```bash
whoami
echo "$HOME"
```

Expected:

```text
<YourUserName>
/home/<YourUserName>
```

Run Google Authenticator:

```bash
google-authenticator \
  --secret="$HOME/.ssh/.google_authenticator"
```

Recommended answers:

```text
Time-based tokens?                         y
Update the secret file?                    y
Disallow multiple uses of the same token?  y
Increase the time-skew window?             n
Enable rate limiting?                      y
```

The employee should scan the QR code using their authenticator application.

Do not copy or share:

- The QR code
- The secret key
- Current TOTP codes
- Emergency scratch codes

The employee should securely store the emergency scratch codes.

Set permissions and restore SELinux labels:

```bash
chmod 600 "$HOME/.ssh/.google_authenticator"
restorecon -v "$HOME/.ssh/.google_authenticator"
```

Verify:

```bash
stat -c '%U:%G %a %n' \
  "$HOME/.ssh/.google_authenticator"

ls -lZ "$HOME/.ssh/.google_authenticator"
```

Exit back to the administrator account:

```bash
exit
```

---

## Step 6: Confirm the User Has No Administrative Access

From the administrator account:

```bash
sudo -l -U <YourUserName>
```

The result should indicate that `<YourUserName>` is not allowed to run commands with `sudo`.

Also verify group membership:

```bash
id <YourUserName>
```

The user should belong to `ssh-access` but not `wheel`.

---

## Step 7: Test the User Login

From the employee's Mac, test with:

```bash
ssh \
  -o IdentitiesOnly=yes \
  -i ~/.ssh/ssh-relay-<YourUserName> \
  <YourUserName>@208.58.24.114
```

Expected authentication sequence:

1. The Mac requests the SSH private-key passphrase.
2. The relay requests a verification code.
3. The employee enters the current TOTP code.
4. Login succeeds.
5. The relay does not request the Linux account password.

After login, run:

```bash
whoami
id
echo "$HOME"
pwd
```

Expected results:

```text
<YourUserName>
/home/<YourUserName>
/home/<YourUserName>
```

The `id` output should include `ssh-access` and should not include `wheel`.

---

## Step 8: Test the Employee's Required Workflow

While logged in through the relay, test outbound SSH:

```bash
ssh user@customer-server
```

Test one known-good local port forward from the Mac:

```bash
ssh \
  -L local_port:destination_host:destination_port \
  -i ~/.ssh/ssh-relay-<YourUserName> \
  <YourUserName>@208.58.24.114
```

Where:

- `local_port` is the port opened on the employee's Mac
- `destination_host` is the remote host reachable from the relay
- `destination_port` is the service port on that remote host

If X11 forwarding is required, test it with:

```bash
ssh \
  -X \
  -i ~/.ssh/ssh-relay-<YourUserName> \
  <YourUserName>@208.58.24.114
```

Review recent SSH logs on the relay:

```bash
sudo journalctl -u sshd --since "10 minutes ago" --no-pager -l
```

A successful login should show accepted public-key authentication followed by successful keyboard-interactive/PAM authentication.

---

# Part 2: Steps for the Employee on Their Mac

## Step 1: Generate an SSH Key

Open Terminal and run:

```bash
ssh-keygen \
  -t ed25519 \
  -a 100 \
  -f ~/.ssh/ssh-relay-<YourUserName>
```

Replace `<YourUserName>` in the filename with the employee's actual relay username.

Enter a strong passphrase when prompted.

This creates:

```text
~/.ssh/ssh-relay-<YourUserName>
~/.ssh/ssh-relay-<YourUserName>.pub
```

The file without `.pub` is the private key and must never be shared.

---

## Step 2: Verify the Public Key

Run:

```bash
ssh-keygen -lf ~/.ssh/ssh-relay-<YourUserName>.pub
```

Display the public key:

```bash
cat ~/.ssh/ssh-relay-<YourUserName>.pub
```

Copy it to the clipboard:

```bash
pbcopy < ~/.ssh/ssh-relay-<YourUserName>.pub
```

Send only the public key to the relay administrator.

Never send:

```text
~/.ssh/ssh-relay-<YourUserName>
```

That is the private key.

---

## Step 3: Add the Key to the macOS Keychain

Run:

```bash
ssh-add \
  --apple-use-keychain \
  ~/.ssh/ssh-relay-<YourUserName>
```

Enter the SSH key passphrase when prompted.

---

## Step 4: Add an SSH Configuration Entry

Open the SSH configuration file:

```bash
nano ~/.ssh/config
```

Add:

```sshconfig
Host ssh-relay-<YourUserName>
    HostName 208.58.24.114
    Port 9324
    User <YourUserName>
    IdentityFile ~/.ssh/ssh-relay-<YourUserName>
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes
```

Replace:

- `ssh-relay-<YourUserName>` with a convenient alias
- `208.58.24.114` with the production hostname or public address when instructed
- `<YourUserName>` with the employee's actual relay username
- The identity filename if a different filename was used

Save and exit Nano:

1. Press `Ctrl+O`
2. Press `Enter`
3. Press `Ctrl+X`

Set secure permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/ssh-relay-<YourUserName>
chmod 644 ~/.ssh/ssh-relay-<YourUserName>.pub
```

---

## Step 5: Enroll the Authenticator Application

During TOTP enrollment, scan the QR code displayed by the administrator or displayed while the account is being configured.

The employee should:

- Create a clearly named authenticator entry
- Confirm the entry produces six-digit time-based codes
- Store emergency scratch codes securely
- Never share the secret or a current code

---

## Step 6: Test the SSH Connection

Connect using the SSH alias:

```bash
ssh ssh-relay-<YourUserName>
```

Expected sequence:

1. The Mac unlocks or requests the private-key passphrase.
2. The relay asks for a verification code.
3. Enter the current six-digit TOTP code.
4. Login succeeds.

A warning about the connection not using a post-quantum key exchange algorithm may appear. This does not mean the connection is unencrypted.

---

## Step 7: Test Local Port Forwarding

Example:

```bash
ssh \
  -L 8080:internal-host.example.com:80 \
  ssh-relay-<YourUserName>
```

This opens local port `8080` on the employee's Mac and forwards it through the relay to port `80` on `internal-host.example.com`.

Keep the SSH session open while using the tunnel.

---

## Step 8: Test Outbound SSH Through the Relay

Connect to the relay:

```bash
ssh ssh-relay-<YourUserName>
```

Then connect from the relay to the customer server:

```bash
ssh customer-user@customer-server
```

The outbound connection will originate from the office network's public IP address.

---

# Administrator Checklist

For each new employee, confirm:

- [ ] Individual Linux account created
- [ ] User added to `ssh-access`
- [ ] User not added to `wheel`
- [ ] Password locked
- [ ] Individual public key installed
- [ ] `.ssh` permissions verified
- [ ] SELinux labels restored
- [ ] Individual TOTP enrolled
- [ ] Emergency scratch codes stored securely
- [ ] Key-plus-TOTP login tested
- [ ] `sudo` access denied
- [ ] Outbound SSH tested
- [ ] Local forwarding tested
- [ ] X11 forwarding tested if required

# Troubleshooting

## Banned addresses
To see the current jail status:  

```bash  
sudo fail2ban-client status sshd
```  

To unban your current address:  

```bash
sudo fail2ban-client set sshd unbanip YOUR.REMOTE.IP
```  

Edit the Fail2ban jail:  

```bash
sudo nano /etc/fail2ban/jail.d/sshd.local
```
Follow login attempts  

```bash
sudo journalctl -u sshd -f
```
Or

```bash
sudo tail -f /var/log/secure
```

## Create User Template

```bash

sudo useradd \
  --create-home \
  --shell /bin/bash \
  --groups ssh-access \
  --comment "SSH relay user" \
  <YourUserName>

getent passwd <YourUserName>
id <YourUserName>
sudo ls -ld /home/<YourUserName>

sudo passwd -l <YourUserName>
sudo passwd -S <YourUserName>

---

sudo install \
  -d \
  -m 700 \
  -o <YourUserName> \
  -g <YourUserName> \
  /home/<YourUserName>/.ssh

sudo install \
  -m 600 \
  -o <YourUserName> \
  -g <YourUserName> \
  /dev/null \
  /home/<YourUserName>/.ssh/authorized_keys

sudo nano /home/<YourUserName>/.ssh/authorized_keys

---

sudo chown -R <YourUserName>:<YourUserName> /home/<YourUserName>/.ssh
sudo chmod 700 /home/<YourUserName>
sudo chmod 700 /home/<YourUserName>/.ssh
sudo chmod 600 /home/<YourUserName>/.ssh/authorized_keys
sudo restorecon -RFv /home/<YourUserName>/.ssh

sudo stat -c '%U:%G %a %n' \
  /home/<YourUserName> \
  /home/<YourUserName>/.ssh \
  /home/<YourUserName>/.ssh/authorized_keys

sudo ls -ldZ /home/<YourUserName>/.ssh
sudo ls -lZ /home/<YourUserName>/.ssh/authorized_keys

sudo -iu <YourUserName>

whoami
echo "$HOME"

---

google-authenticator \
  --secret="$HOME/.ssh/.google_authenticator"

---

chmod 600 "$HOME/.ssh/.google_authenticator"
restorecon -v "$HOME/.ssh/.google_authenticator"

stat -c '%U:%G %a %n' \
  "$HOME/.ssh/.google_authenticator"

ls -lZ "$HOME/.ssh/.google_authenticator"



```