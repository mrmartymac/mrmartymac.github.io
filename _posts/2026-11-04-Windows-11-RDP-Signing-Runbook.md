---
title: Signing RDP Shortcut
date: 2026-11-04 12:37:32 -400
categories: [Cheatsheets, RDP]
tags: [rdp, certificate, warning] # TAG names should always be lowercase
author: mm
---

# Windows 11 Pro: sign and trust a saved .rdp file

Prepared: November 4, 2026. Source: the **Stop RDP Warning** conversation, ID `6aeb43fc-a87c-83ea-b9f1-9885d579e69a`.

## Purpose and evidence

Repeat the workstation procedure that successfully signed `SuppAlere.rdp` and resolved the reported publisher warning. This concerns the saved file's publisher; it does not establish the identity or security of the remote server.

Evidence labels used below:

- **Verified in transcript:** actual command/output or an explicit user report.
- **Instructed in successful sequence:** directions given before the final success report, without separate output proving each action.
- **Additional guidance / inferred:** repeatability improvements and diagnostic suggestions, not independently tested in the conversation.

| Item | Evidence |
| --- | --- |
| Environment | Windows 11 Pro per the requested runbook; workstation/account output identifies `KIM-PC`, administrator `scsadmin`, and standard desktop user `kbrown`. Exact Windows build was not captured. |
| Certificate creation | Verified: legacy provider, RSA 2048, SHA256, signature key, Code Signing EKU, and private key present. |
| Initial failure | Verified: legacy certificate still produced `0x80070490`; changing provider alone did not resolve it. |
| Separate signing test | Verified: PowerShell signed a test script, but validation reported an untrusted root. |
| Public certificate export and machine trust imports | Instructed in successful sequence; import output and subsequent `Valid` test output were not included. |
| Final .rdp signing | Verified: `Test-Path` returned `True`, `rdpsign` reported success, and signature fields appeared. |
| Trusted publisher policy and `gpupdate` | Instructed immediately before the final success report; exact applied policy value/output was not captured. |
| End result | Verified user report: “It works! Thank you.” No separate screenshot of the resulting launch was supplied. |
| Error `0x80092004` | Requested for this runbook, but not present in the retrievable turns. Its diagnostics below are additional guidance. |

The reader returned no older-page cursor. Do not treat missing intermediate output as proof that a particular change alone fixed the problem.

## 1. Identify the administrator and the user's actual file

Open **Windows PowerShell as administrator**, using the account that will hold the signing key. In the tested case, the desktop was logged in as `kbrown`, but elevation used `scsadmin` credentials.

```powershell
whoami
$env:USERPROFILE
```

The transcript showed `kim-pc\scsadmin`. In that elevated window:

- `Cert:\CurrentUser\My` means **scsadmin's** Personal certificate store.
- `$env:USERPROFILE` and `$env:TEMP` refer to **scsadmin**, not kbrown.
- Run later signing commands as this same administrator; the private key remains with that account.
- Test the final connection from the normal **kbrown** desktop.

Use an explicit file path:

```powershell
$rdp = 'C:\Users\kbrown\Desktop\SuppAlere.rdp'
if (-not (Test-Path -LiteralPath $rdp -PathType Leaf)) {
    throw 'RDP file not found. Check the standard user desktop path.'
}
```

**Additional guidance:** desktops may be redirected to OneDrive or another folder. Locate the actual `.rdp` file, not merely a `.lnk` shortcut. Back it up before signing, since signing overwrites it:

```powershell
$backup = $rdp + '.' + (Get-Date -Format 'yyyyMMdd-HHmmss') + '.bak'
Copy-Item -LiteralPath $rdp -Destination $backup
```

## 2. Create the legacy CryptoAPI code signing certificate

**Verified command**, expressed with a parameter table for easier copying. Run once for a new signing identity. If repeating on the original workstation with its existing key, skip creation and load the existing certificate below.

```powershell
$certParams = @{
    Subject           = 'CN=SCS RDP Publisher Legacy'
    Type              = 'CodeSigningCert'
    Provider          = 'Microsoft Enhanced RSA and AES Cryptographic Provider'
    KeySpec           = 'Signature'
    KeyAlgorithm      = 'RSA'
    KeyLength         = 2048
    HashAlgorithm     = 'SHA256'
    CertStoreLocation = 'Cert:\CurrentUser\My'
    NotAfter          = (Get-Date).AddYears(3)
}
$cert = New-SelfSignedCertificate @certParams
$thumbprint = $cert.Thumbprint

$cert | Format-List Subject, Thumbprint, HasPrivateKey, NotBefore, NotAfter
$cert.EnhancedKeyUsageList | Format-Table FriendlyName, ObjectId
certutil.exe -user -store My $thumbprint
```

Expected checks from the tested certificate:

- `HasPrivateKey : True`.
- Code Signing EKU: `1.3.6.1.5.5.7.3.3`.
- Key usage: Digital Signature.
- Provider: `Microsoft Enhanced RSA and AES Cryptographic Provider`.
- `certutil` reported that its signature test passed.

This is a legacy **CryptoAPI CSP**, rather than the CNG `Microsoft Software Key Storage Provider` shown for an earlier certificate. The final successful certificate used the legacy provider. The evidence does **not** establish that all CNG certificates are incompatible with `rdpsign`.

To reuse the original certificate in a new elevated PowerShell session under `scsadmin`:

```powershell
$thumbprint = 'D9574F41676F8BF0481A14A22A8E38FEA5B610AB'
$cert = Get-Item "Cert:\CurrentUser\My\$thumbprint" -ErrorAction Stop
```

For a newly created certificate, always use **its own** `$cert.Thumbprint`. The historical thumbprint above will not identify a newly generated certificate.

### SHA-1 thumbprint versus `/sha256`

The successful command used `/sha256` with the certificate's **40-character SHA-1 thumbprint**, obtained from `$cert.Thumbprint` / `Cert Hash(sha1)`:

```text
D9574F41676F8BF0481A14A22A8E38FEA5B610AB
```

Keep that tested combination. Do not substitute the hash of the `.rdp` file or `.cer` file. Remove spaces and invisible characters when copying thumbprints; taking the value directly from `$cert.Thumbprint` avoids copy/paste artifacts.

**Documentation caveat:** Microsoft's current `rdpsign` page describes the `/sha256` argument as a SHA256 certificate hash. That wording differs from this observed successful use of the SHA-1 thumbprint. This runbook preserves the observed behavior; it does not claim the documentation proves that behavior on every Windows build. See [Microsoft's rdpsign reference](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/rdpsign).

## 3. Export the public certificate and trust it for all local users

**Instructed in successful sequence.** Stay in the same elevated administrator session:

```powershell
$cerFile = Join-Path $env:TEMP 'SCS-RDP-Publisher.cer'
Export-Certificate -Cert $cert -FilePath $cerFile -ErrorAction Stop

Import-Certificate -FilePath $cerFile -CertStoreLocation 'Cert:\LocalMachine\Root' -ErrorAction Stop
Import-Certificate -FilePath $cerFile -CertStoreLocation 'Cert:\LocalMachine\TrustedPublisher' -ErrorAction Stop
```

`Export-Certificate` exports the public certificate only. Do not distribute a PFX/private key to the standard user. Machine-level imports make trust available to both the administrator and standard user on this workstation; they do not copy the private signing key to kbrown.

**Additional verification:** confirm the same thumbprint exists in both stores:

```powershell
Get-Item "Cert:\LocalMachine\Root\$thumbprint" |
    Format-List Subject, Thumbprint, NotAfter
Get-Item "Cert:\LocalMachine\TrustedPublisher\$thumbprint" |
    Format-List Subject, Thumbprint, NotAfter
```

Trusting this self-signed certificate in LocalMachine Root is a machine-wide trust decision. Verify the certificate's identity before importing it, and protect the administrator's signing key. Trust is associated with the certificate, not just this single `.rdp` file.

## 4. Check key operation and certificate trust

The transcript used a harmless script as a diagnostic; it need not be executed:

```powershell
$testFile = Join-Path $env:TEMP 'rdp-sign-test.ps1'
Set-Content -LiteralPath $testFile -Value '# RDP certificate signing test'
Set-AuthenticodeSignature -FilePath $testFile -Certificate $cert
Get-AuthenticodeSignature -FilePath $testFile |
    Format-List Status, StatusMessage, SignerCertificate
```

Before trust imports, the **verified** output was `UnknownError`, with a message that the chain ended in an untrusted root. The signer certificate was the expected legacy certificate. That showed a signature had been applied while trust validation failed.

After trust imports, the **expected, but not separately captured**, result is `Status : Valid` and `StatusMessage : Signature verified.` Investigate any other result before continuing. This test checks a PowerShell signature; it is not a cryptographic verification of the `.rdp` file.

## 5. Sign the standard user's saved .rdp file

Run under the administrator that owns the signing key. Re-establish the explicit path if this is a new session:

```powershell
$rdp = 'C:\Users\kbrown\Desktop\SuppAlere.rdp'
Test-Path -LiteralPath $rdp
rdpsign.exe /sha256 $thumbprint /v $rdp
```

The transcript's **verified successful command** was:

```powershell
rdpsign.exe /sha256 D9574F41676F8BF0481A14A22A8E38FEA5B610AB /v $rdp
```

It reported:

```text
All rdp file(s) have been succesfully signed.
```

Check the signature fields:

```powershell
Get-Content -LiteralPath $rdp | Select-String '^signscope:s:|^signature:s:'
```

Both fields appeared in the successful transcript. Their presence establishes that signing data exists, but does not by itself prove that the current file's signature is valid or trusted. The standard-user launch test below is essential. Do not reconstruct a signature by copying the chat's displayed signature text.

## 6. Configure the trusted .rdp publisher policy

**Instructed in successful sequence**, followed by the user's success report.

1. Open `gpedit.msc` as administrator.
2. Navigate to **Computer Configuration → Administrative Templates → Windows Components → Remote Desktop Services → Remote Desktop Connection Client**.
3. Open **Specify SHA1 thumbprints of certificates representing trusted .rdp publishers**.
4. Select **Enabled**.
5. Enter the signing certificate's thumbprint. The instructed historical value was:

   ```text
   D9574F41676F8BF0481A14A22A8E38FEA5B610AB
   ```

6. Select **Apply**, then **OK**.

**Format caveat:** inspect the installed policy's Help pane. For the SHA1 policy above, use the raw 40-character thumbprint without spaces, quotes, or an invented prefix. If that machine's updated template explicitly specifies another hash or a prefix, follow its documented format and validate the result. The conversation did not capture such a variant; it does not establish that `sha1:` or `sha256:` prefixes are needed. For multiple publishers, follow the local Help pane's separator requirements; one publisher avoids that ambiguity.

Microsoft documents this policy as accepting SHA1 certificate thumbprints and ignoring invalid strings. See [trusted .rdp publisher policy](https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-admx-terminalserver#ts_client_trusted_certificate_thumbprints_1).

Use the **Computer Configuration** policy for this procedure. Setting a user policy while running as scsadmin may target the wrong user's settings. In managed environments, domain policy may override local settings; inspect effective policy rather than assuming the local editor is authoritative.

## 7. Refresh policy and test as the standard user

In elevated PowerShell:

```powershell
gpupdate.exe /force
```

Then close the elevated window, return to the normal `kbrown` desktop, and double-click the **same signed** `SuppAlere.rdp` file. Close any earlier publisher dialog and launch it afresh.

The acceptance criterion for the original case was that the publisher warning disappeared; the user reported success after these directions. Confirm the intended destination and publisher, and distinguish any remaining server certificate, credential, or redirection-consent dialog from the saved-file publisher warning. Windows builds and organizational policies may display additional prompts.

**Additional guidance:** if policy does not appear to apply, inspect the effective computer policy from an elevated prompt:

```powershell
gpresult.exe /scope computer /r
```

## Troubleshooting

### `0x80092004` — `CRYPT_E_NOT_FOUND`

Microsoft identifies this as a cryptographic object/property lookup failure. The error number alone does not identify the missing object. See [Microsoft COM error codes](https://learn.microsoft.com/en-us/windows/win32/com/com-error-codes-4).

**Additional guidance; occurrence and exact fix are not verified in the retrieved conversation:**

1. Confirm the command is running under the account that owns the certificate and private key (`whoami`).
2. Check the selected thumbprint exists in that account's Personal store and has a private key.
3. Use `$cert.Thumbprint`, not a copied hash with spaces, an old certificate's thumbprint, or a file hash.
4. Check provider, Code Signing EKU, validity dates, and the script diagnostic in step 4.
5. If reproducing this runbook with an unsuitable earlier certificate, create the legacy CSP certificate in step 2, then export, trust, configure policy, and sign using the **new** thumbprint. Changing provider alone is not a proven universal fix.

Useful checks for either signing error:

```powershell
whoami
$cert = Get-Item "Cert:\CurrentUser\My\$thumbprint" -ErrorAction Stop
$cert | Format-List Subject, Thumbprint, HasPrivateKey, NotBefore, NotAfter
$cert.EnhancedKeyUsageList | Format-Table FriendlyName, ObjectId
certutil.exe -user -store My $thumbprint
Test-Path -LiteralPath $rdp
```

### `0x80070490` — `ERROR_NOT_FOUND` / element not found

This is a general lookup error, not proof that the certificate or private key is absent. Microsoft's [error reference example](https://learn.microsoft.com/en-us/troubleshoot/windows-server/installing-updates-features-roles/troubleshoot-windows-update-error-0x80070490) identifies the code; its Windows Update repair steps are not applicable to this RDP procedure.

**Verified in this case:** the legacy certificate existed, had a private key, had the expected EKU and provider, and passed `certutil`'s signature test, yet `rdpsign` still failed. A script signature then exposed an untrusted root. The next directions imported the public certificate to the machine trust stores and corrected the file path to kbrown's desktop; the next posted `rdpsign` attempt succeeded.

Check both trust and the explicit file path. The earlier `$env:USERPROFILE\Desktop` expression referred to **scsadmin's** desktop. Because trust and path were addressed before the successful attempt, their individual causal contributions were not isolated.

Optional diagnostic supported by `rdpsign`:

```powershell
rdpsign.exe /sha256 $thumbprint /l /v $rdp
```

`/l` tests signing without replacing the input file. It also failed in the earlier transcript, so it is a diagnostic, not a repair. If troubleshooting continues, retain the full output, account name, Windows build, provider, validity, and store locations before creating more certificates.

### Still showing a publisher warning

**Additional guidance:** verify all of the following:

- You signed the actual file being launched, rather than another desktop copy.
- It has not been edited since signing, and both signature fields are present.
- The public certificate with that exact thumbprint is in **LocalMachine Root** and **LocalMachine TrustedPublisher** on the computer launching the file.
- The trusted publisher policy contains the current certificate's thumbprint in the accepted format and survives `gpupdate /force`.
- Effective domain/management policy does not replace the local setting.
- The certificate is within its validity period and the system clock is correct.
- The dialog concerns the file publisher; signing does not fix a remote server's TLS certificate/name mismatch or suppress every RDP prompt.

## Maintenance, expiration, and security

- **Re-sign after edits.** Finish changing connection settings first, run `rdpsign` again under the signing administrator, then repeat the normal-user launch test. Do not assume saving through Remote Desktop Connection preserves a valid signature.
- **Original expiration:** the tested certificate's reported `NotAfter` was **November 4, 2029, 10:57:45 AM**, as displayed on that workstation. Check `$cert.NotAfter`; new certificates have their own dates.
- **Renew before expiry.** Create a replacement certificate, deploy its public trust, update the trusted publisher policy, re-sign all affected `.rdp` files, and test. Remove retired trust only after accounting for other files using it. Renewal was not tested in the conversation.
- **Do not assume a lasting timestamp.** This procedure did not configure or verify timestamping. Plan to replace signatures before the signing certificate expires; post-expiration behavior was not tested.
- **Protect the private key.** Keep it with the designated signing account. A `.cer` is public and cannot restore a lost signing key. Any private-key backup requires a separate, secured process; PFX export was not part of the tested procedure.
- **Limit trust.** A self-signed certificate added to machine Root becomes a local trust anchor, and TrustedPublisher has effects beyond this one connection. Do not import unknown certificates or reuse one private key broadly across unrelated customers. An organizational PKI or managed signing process may be preferable for broader deployment; that alternative was not tested here.
- **Review connection permissions.** A trusted publisher can sign files that request clipboard, drive, printer, or other redirection. Review the intended destination and redirection settings before signing; signatures do not establish that those settings are safe.
- **Preserve server authentication.** Do not weaken NLA, TLS validation, or `authentication level` to silence unrelated warnings. No such weakening is required by this runbook.

## References and scope of validation

The successful chat is the source of the specific account, certificate, path, command, output, and user acceptance evidence. The commands were **not re-executed on Windows while preparing this document**. Backups, explicit error stops, store checks, effective-policy checks, renewal advice, and uncaptured error diagnostics are additional operational guidance.

- [Microsoft: rdpsign](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/rdpsign) — command behavior and the documentation caveat noted above.
- [Microsoft: New-SelfSignedCertificate](https://learn.microsoft.com/en-us/powershell/module/pki/new-selfsignedcertificate) — certificate parameters; Microsoft describes this cmdlet as creating self-signed certificates for testing.
- [Microsoft: trusted .rdp publisher policy](https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-admx-terminalserver#ts_client_trusted_certificate_thumbprints_1) — policy location and thumbprint semantics.
- [Microsoft: COM security error codes](https://learn.microsoft.com/en-us/windows/win32/com/com-error-codes-4) — `0x80092004` meaning.
