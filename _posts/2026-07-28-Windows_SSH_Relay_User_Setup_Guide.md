---
title: Windows SSH Relay User Setup Guide
date: 2026-07-28 08:39:18 -400
categories: [Documentation, SSH-Relays]
tags: [proxmox, ssh, relay] # TAG names should always be lowercase
author: mm
---


This guide explains how a new Windows user can configure the built-in OpenSSH client to connect to the SSH relay using:

- An individual SSH key
- A private-key passphrase
- TOTP verification
- The SSH alias `ssh-relay-2`
- Local port forwarding with `ssh -L`

No additional SSH software is required. These steps use PowerShell and the built-in Windows OpenSSH client.

Replace `<YourUserName>` with the Linux username assigned by the SSH relay administrator.

---

# 1. Confirm the OpenSSH Client Is Installed

Open PowerShell and run:

```powershell
ssh -V
```

A successful result should begin with something similar to:

```text
OpenSSH_for_Windows
```

Also verify that the required commands are available:

```powershell
Get-Command ssh
Get-Command ssh-keygen
```

They will normally be located under:

```text
C:\Windows\System32\OpenSSH\
```

If `ssh` is not found, open PowerShell as Administrator and run:

```powershell
Get-WindowsCapability -Online |
    Where-Object Name -Like 'OpenSSH.Client*'
```

If the client is not installed, install it with:

```powershell
Add-WindowsCapability `
    -Online `
    -Name OpenSSH.Client~~~~0.0.1.0
```

Only the OpenSSH Client is required. The OpenSSH Server feature is not needed.

---

# 2. Generate an SSH Key Pair

In PowerShell, run:

```powershell
ssh-keygen `
    -t ed25519 `
    -a 100 `
    -f "$HOME\.ssh\ssh-relay-<YourUserName>"
```

Example:

```powershell
ssh-keygen `
    -t ed25519 `
    -a 100 `
    -f "$HOME\.ssh\ssh-relay-mrmar"
```

Enter a strong passphrase when prompted.

This creates two files:

```text
C:\Users\<WindowsUser>\.ssh\ssh-relay-<YourUserName>
C:\Users\<WindowsUser>\.ssh\ssh-relay-<YourUserName>.pub
```

The file ending in `.pub` is the public key.

The file without `.pub` is the private key and must never be shared.

---

# 3. Verify and Copy the Public Key

Verify the public key fingerprint:

```powershell
ssh-keygen -lf "$HOME\.ssh\ssh-relay-<YourUserName>.pub"
```

Display the public key:

```powershell
Get-Content "$HOME\.ssh\ssh-relay-<YourUserName>.pub"
```

Copy it directly to the Windows clipboard:

```powershell
Get-Content "$HOME\.ssh\ssh-relay-<YourUserName>.pub" |
    Set-Clipboard
```

Send only the public key to the SSH relay administrator.

It will look similar to:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... WindowsUser@ComputerName
```

Never send this file:

```text
C:\Users\<WindowsUser>\.ssh\ssh-relay-<YourUserName>
```

That is the private key.

---

# 4. Wait for the Administrator to Complete Server Setup

The administrator must:

- Create the Linux account
- Add the account to the `ssh-access` group
- Install the public key in `authorized_keys`
- Enroll the user for TOTP authentication
- Confirm that the account has no unintended administrative access

Do not attempt the final login until the administrator confirms that server-side setup is complete.

---

# 5. Create the Windows SSH Configuration

Open the SSH configuration file:

```powershell
notepad "$HOME\.ssh\config"
```

Add the following entry:

```sshconfig
Host ssh-relay-2
    HostName 208.58.24.114
    Port 9324
    User <YourUserName>
    IdentityFile ~/.ssh/ssh-relay-<YourUserName>
    IdentitiesOnly yes
```

Example:

```sshconfig
Host ssh-relay-2
    HostName 208.58.24.114
    Port 9324
    User mrmar
    IdentityFile ~/.ssh/ssh-relay-mrmar
    IdentitiesOnly yes
```

Save the file.

The SSH configuration file should be located at:

```text
C:\Users\<WindowsUser>\.ssh\config
```

---

# 6. Verify the SSH Configuration

Run:

```powershell
ssh -G ssh-relay-2 |
    Select-String '^(hostname|user|port|identityfile|identitiesonly) '
```

Expected values should resemble:

```text
hostname 208.58.24.114
user <YourUserName>
port 9324
identityfile ~/.ssh/ssh-relay-<YourUserName>
identitiesonly yes
```

Confirm that the actual username and key filename appear, rather than the placeholder `<YourUserName>`.

---

# 7. Connect to the SSH Relay

Run:

```powershell
ssh ssh-relay-2
```

The expected authentication sequence is:

1. Windows requests the SSH private-key passphrase.
2. The relay requests a verification code.
3. Enter the current six-digit TOTP code.
4. Login succeeds.
5. The Linux account password should not be requested.

After login, verify the account:

```bash
whoami
id
echo "$HOME"
pwd
```

The results should show the assigned relay username and home directory.

The `id` output should include the `ssh-access` group.

---

# 8. Handle a Changed Host-Key Warning

If the relay replaces an older server using the same public address and port, Windows may display:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Do not ignore this warning without verifying the new key.

Ask the administrator to confirm the relay's ED25519 fingerprint.

Once the fingerprint has been verified, remove the old saved host key:

```powershell
ssh-keygen -R "[208.58.24.114]:9324"
```

Reconnect:

```powershell
ssh ssh-relay-2
```

Windows will ask whether to trust the new host key.

Confirm that the displayed fingerprint matches the fingerprint supplied by the administrator, then enter:

```text
yes
```

The new key will be saved in:

```text
C:\Users\<WindowsUser>\.ssh\known_hosts
```

---

# 9. Test Local Port Forwarding

Local port forwarding uses the `-L` option.

Example:

```powershell
ssh -L 8080:destination.example.com:80 ssh-relay-2
```

This forwards:

```text
Windows PC localhost:8080
        ↓
SSH relay
        ↓
destination.example.com:80
```

Keep the SSH session open while using the tunnel.

To create the tunnel without opening an interactive shell:

```powershell
ssh -N -L 8080:destination.example.com:80 ssh-relay-2
```

Where:

- `8080` is the local port on the Windows PC
- `destination.example.com` is the destination reachable from the relay
- `80` is the destination service port

---

# 10. Save a Frequently Used Port Forward

A frequently used tunnel can be stored in the SSH configuration file.

Example:

```sshconfig
Host customer-web-tunnel
    HostName 208.58.24.114
    Port 9324
    User <YourUserName>
    IdentityFile ~/.ssh/ssh-relay-<YourUserName>
    IdentitiesOnly yes
    LocalForward 8080 destination.example.com:80
```

Start the tunnel with:

```powershell
ssh -N customer-web-tunnel
```

---

# 11. SSH to a Customer Server Through the Relay

Connect to the relay:

```powershell
ssh ssh-relay-2
```

Then, from the relay:

```bash
ssh customer-user@customer-server
```

The connection to the customer server will originate from the office network's known public IP address.

---

# 12. Optional Verbose Troubleshooting

For detailed connection diagnostics:

```powershell
ssh -vvv ssh-relay-2
```

Useful lines include:

```text
Offering public key
Server accepts key
Authentications that can continue
Next authentication method
Verification code
Permission denied
Connection closed
```

To view only the most relevant lines:

```powershell
ssh -vvv ssh-relay-2 2>&1 |
    Select-String 'Connecting to|identity file|Offering public key|Server accepts key|Authentications that can continue|Next authentication method'
```

A successful key exchange should include:

```text
Server accepts key
```

After that, the next authentication method should be keyboard-interactive or a verification-code prompt.

---

# 13. Security Requirements

The user must:

- Keep the private key only on the assigned Windows PC
- Protect the private key with a passphrase
- Never email, upload, or copy the private key to the relay
- Never share a current TOTP code
- Store emergency TOTP recovery codes securely
- Report a lost or replaced PC immediately
- Report an unexpected host-key warning before accepting a new key
- Use only their assigned individual account

---

# Windows User Checklist

- [ ] OpenSSH Client confirmed
- [ ] Individual SSH key generated
- [ ] Private key protected with a passphrase
- [ ] Public key sent to the administrator
- [ ] SSH configuration created
- [ ] `ssh-relay-2` alias verified
- [ ] Host fingerprint verified
- [ ] TOTP enrolled
- [ ] Key-plus-TOTP login tested
- [ ] Local forwarding tested if required
- [ ] Outbound SSH through the relay tested
