---
title: Mac SSH Relay User Setup Guide
date: 2026-07-28 08:39:18 -400
categories: [Documentation, SSH-Relays]
tags: [proxmox, ssh, relay] # TAG names should always be lowercase
author: mm
---

This guide explains how a Mac user can configure the built-in OpenSSH client to connect to the SSH relay using an individual SSH key, TOTP verification, the `ssh-relay-2` alias, and optional port forwarding.

Replace `<YourUserName>` with the Linux username assigned by the SSH relay administrator.

---

## 1. Confirm SSH Is Available

Open Terminal and run:

```bash
ssh -V
which ssh
which ssh-keygen
```

macOS includes OpenSSH by default.

---

## 2. Create the `.ssh` Directory

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

---

## 3. Generate an SSH Key Pair

```bash
ssh-keygen \
  -t ed25519 \
  -a 100 \
  -f ~/.ssh/ssh-relay-<YourUserName>
```

Enter a strong passphrase when prompted.

This creates:

```text
~/.ssh/ssh-relay-<YourUserName>
~/.ssh/ssh-relay-<YourUserName>.pub
```

The `.pub` file is the public key. The file without `.pub` is the private key and must never be shared.

---

## 4. Copy the Public Key

Verify its fingerprint:

```bash
ssh-keygen -lf ~/.ssh/ssh-relay-<YourUserName>.pub
```

Display it:

```bash
cat ~/.ssh/ssh-relay-<YourUserName>.pub
```

Copy it to the clipboard:

```bash
pbcopy < ~/.ssh/ssh-relay-<YourUserName>.pub
```

Send only the public key to the SSH relay administrator.

---

## 5. Wait for Server-Side Setup

The administrator must:

- Create the Linux account
- Add it to `ssh-access`
- Install the public key in `authorized_keys`
- Enroll the user for TOTP
- Verify that the account has no unintended sudo access

---

## 6. Add the Private Key to macOS Keychain

```bash
ssh-add \
  --apple-use-keychain \
  ~/.ssh/ssh-relay-<YourUserName>
```

Enter the private-key passphrase when prompted.

---

## 7. Create the SSH Configuration

Open:

```bash
nano ~/.ssh/config
```

Add:

```sshconfig
Host ssh-relay-2
    HostName 208.58.24.114
    Port 9324
    User <YourUserName>
    IdentityFile ~/.ssh/ssh-relay-<YourUserName>
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes
```

Save with `Ctrl+O`, press `Enter`, then exit with `Ctrl+X`.

Set permissions:

```bash
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/ssh-relay-<YourUserName>
chmod 644 ~/.ssh/ssh-relay-<YourUserName>.pub
```

---

## 8. Verify the SSH Configuration

```bash
ssh -G ssh-relay-2 |
grep -Ei '^(hostname|user|port|identityfile|identitiesonly) '
```

Expected values should include:

```text
hostname 208.58.24.114
user <YourUserName>
port 9324
identityfile ~/.ssh/ssh-relay-<YourUserName>
identitiesonly yes
```

---

## 9. Connect to the Relay

```bash
ssh ssh-relay-2
```

Expected sequence:

1. macOS unlocks the private key or asks for its passphrase.
2. The relay asks for a verification code.
3. Enter the current six-digit TOTP code.
4. Login succeeds.
5. The Linux password should not be requested.

After login:

```bash
whoami
id
echo "$HOME"
pwd
```

The `id` output should include `ssh-access`.

---

## 10. Handle a Changed Host-Key Warning

If macOS reports:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

first verify the new ED25519 fingerprint with the administrator.

Then remove the old key:

```bash
ssh-keygen -R "[208.58.24.114]:9324"
```

Reconnect:

```bash
ssh ssh-relay-2
```

Confirm the displayed fingerprint matches the administrator's fingerprint, then enter:

```text
yes
```

---

## 11. Test Local Port Forwarding

Example:

```bash
ssh -L 8080:destination.example.com:80 ssh-relay-2
```

This forwards:

```text
Mac localhost:8080
        ↓
SSH relay
        ↓
destination.example.com:80
```

For a tunnel without an interactive shell:

```bash
ssh -N -L 8080:destination.example.com:80 ssh-relay-2
```

Keep the SSH process running while using the tunnel.

---

## 12. Save a Frequently Used Tunnel

Add another entry to `~/.ssh/config`:

```sshconfig
Host customer-web-tunnel
    HostName 208.58.24.114
    Port 9324
    User <YourUserName>
    IdentityFile ~/.ssh/ssh-relay-<YourUserName>
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes
    LocalForward 8080 destination.example.com:80
```

Start it with:

```bash
ssh -N customer-web-tunnel
```

---

## 13. SSH to a Customer Server Through the Relay

Connect to the relay:

```bash
ssh ssh-relay-2
```

Then, from the relay:

```bash
ssh customer-user@customer-server
```

The customer server should see the office's public IP address.

Verify the relay's outbound public IP:

```bash
curl https://api.ipify.org
echo
```

---

## 14. Optional: Route Mac Traffic Through the Office with `sshuttle`

Ordinary SSH does not change the public IP used by applications running locally on the Mac.

Install `sshuttle` with Homebrew:

```bash
brew install sshuttle
```

Start it using the SSH alias:

```bash
sudo sshuttle \
  --dns \
  -r ssh-relay-2 \
  0/0
```

Keep that Terminal window open.

In another Terminal window, verify:

```bash
curl https://api.ipify.org
echo
```

It should show the office public IP.

Stop `sshuttle` with `Ctrl+C`.

---

## 15. Troubleshoot a Timeout

```bash
ssh -vvv \
  -o ConnectTimeout=10 \
  ssh-relay-2
```

If it stops at:

```text
Connecting to 208.58.24.114 port 9324
Operation timed out
```

the problem occurs before SSH authentication.

Possible causes:

- Fail2ban has banned the remote public IP
- The firewall port-forward rule is not matching
- The relay is unreachable
- NAT loopback is unavailable when testing from inside the office

Determine the Mac's current public IP:

```bash
curl https://api.ipify.org
echo
```

Give that address to the administrator for a Fail2ban check.

---

## 16. Troubleshoot Too Many Authentication Failures

Confirm the SSH configuration contains:

```sshconfig
IdentityFile ~/.ssh/ssh-relay-<YourUserName>
IdentitiesOnly yes
```

Then use:

```bash
ssh ssh-relay-2
```

To force the correct key explicitly:

```bash
ssh \
  -o IdentitiesOnly=yes \
  -i ~/.ssh/ssh-relay-<YourUserName> \
  -p 9324 \
  <YourUserName>@208.58.24.114
```

---

## 17. Post-Quantum Warning

Newer macOS OpenSSH clients may report that the connection is not using a post-quantum key-exchange algorithm.

This does not mean the SSH connection is unencrypted. Continue only after confirming the host fingerprint is correct.

---

## 18. Security Requirements

- Keep the private key only on the assigned Mac.
- Protect it with a strong passphrase.
- Never email, upload, or copy the private key to the relay.
- Never share a current TOTP code.
- Store emergency recovery codes securely.
- Report a lost or replaced Mac immediately.
- Verify unexpected host-key changes with the administrator.
- Use only the assigned individual account.

---

## Mac User Checklist

- [ ] SSH client confirmed
- [ ] Individual key generated
- [ ] Private key protected with a passphrase
- [ ] Public key sent to the administrator
- [ ] Private key added to macOS Keychain
- [ ] SSH configuration created
- [ ] `ssh-relay-2` alias verified
- [ ] Host fingerprint verified
- [ ] TOTP enrolled
- [ ] Key-plus-TOTP login tested
- [ ] Local forwarding tested if required
- [ ] Outbound SSH through the relay tested
- [ ] `sshuttle` tested if office-IP routing is required
