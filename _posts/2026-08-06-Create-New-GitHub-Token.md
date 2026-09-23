---
title: Annual Forgejo GitHub Mirror Token Rotation Guide
date: 2026-08-06 19:57:51 -400
categories: [Documentation, Forgejo]
tags: [token, forgejo, yearly] # TAG names should always be lowercase
author: mm
---


## Purpose

> **Copy buttons:** When this Markdown is rendered by Forgejo, GitHub, or another compatible viewer, each fenced command or text block should display a Copy button. A self-contained HTML edition with built-in Copy buttons is also provided alongside this file.

This guide documents the annual process for replacing the GitHub personal access token used by the Forgejo pull mirrors on the Proxmox Forgejo container.

The current mirror direction is:

```text
GitHub → Forgejo
```

Forgejo is configured as a one-way pull mirror for repositories in the GitHub organization:

```text
newspapersystems
```

The Forgejo service runs under the Linux account:

```text
git
```

The GitHub account currently used for mirror authentication is:

```text
mrmartymac
```

Forgejo version at the time this guide was written:

```text
Forgejo 15.0.2
```

In this Forgejo version, the REST API can update the mirror interval but cannot update the stored username and password for existing pull mirrors. The token therefore must be updated in the Git remote configuration for each mirrored bare repository.

---

## Summary of the Annual Process

1. Create a new GitHub personal access token.
2. Test the token directly against one private repository.
3. Test the token against one Forgejo mirror.
4. Run the bulk-update script in dry-run mode.
5. Run the bulk-update script in apply mode.
6. Verify that Forgejo mirrors synchronize successfully.
7. Retain the backup configuration files temporarily.
8. Remove old backup files after the mirrors have been stable.

---

## Important Security Notes

- Do not place the GitHub token directly into the script.
- Do not paste the token into documentation, tickets, email, or chat.
- The script prompts for the token without displaying it.
- Avoid displaying the full Git remote URL after the update because the URL contains the token.
- Run these steps as `root` inside the Forgejo container.
- Revoke the old GitHub token only after the new token has been tested successfully.

---

# Part 1 — Create the New GitHub Token

Sign in to GitHub using the account that Forgejo uses for mirror access.

Current account:

```text
mrmartymac
```

Navigate to:

```text
GitHub
→ Profile picture
→ Settings
→ Developer settings
→ Personal access tokens
→ Fine-grained tokens
```

Create a new token with access to the required repositories in the `newspapersystems` organization.

At minimum, the token needs read access to repository contents.

Recommended settings:

```text
Resource owner: newspapersystems
Repository access: All required mirrored repositories
Contents permission: Read-only
Expiration: Use the longest period allowed by company policy
```

Copy the token immediately. GitHub will not display it again.

Do not revoke the old token yet.

---

# Part 2 — Enter the Forgejo Container

From the Proxmox host:

```bash
pct enter <FORGEJO_CT_ID>
```

Replace `<FORGEJO_CT_ID>` with the actual container ID.

Confirm that you are inside the Forgejo container:

```bash
hostname
```

---

# Part 3 — Test the New GitHub Token

Run:

```bash
read -rp "GitHub username: " GH_USER
read -rsp "GitHub token: " GH_TOKEN
echo

git -c credential.helper= \
    -c core.askPass= \
    ls-remote \
    "https://${GH_USER}:${GH_TOKEN}@github.com/newspapersystems/indexservice.git"

unset GH_TOKEN GH_USER
```

Enter:

```text
GitHub username: mrmartymac
GitHub token: <paste the new token>
```

A successful test returns commit hashes and references similar to:

```text
827bca8b22a163575026edb48cef4f9753cadacc    HEAD
827bca8b22a163575026edb48cef4f9753cadacc    refs/heads/master
```

If the test fails, do not continue.

Common failure messages include:

```text
Invalid username or token
Repository not found
Resource not accessible by personal access token
```

Check the token permissions, repository selection, organization approval, and username.

---

# Part 4 — Test One Forgejo Mirror

## Locate the Repository

Run:

```bash
find /var/lib/forgejo -type d \
  -path '*/newspapersystems/indexservice.git' \
  -print
```

Set the returned path:

```bash
REPO_PATH="/var/lib/forgejo/data/forgejo-repositories/newspapersystems/indexservice.git"
```

Adjust the path if the `find` command returned a different location.

## Back Up the Repository Configuration

```bash
cp -a "$REPO_PATH/config" \
  "$REPO_PATH/config.before-token-update"
```

## Update the Remote for the Test Repository

```bash
GH_USER="mrmartymac"
read -rsp "New GitHub token: " GH_TOKEN
echo

runuser -u git -- \
  git --git-dir="$REPO_PATH" remote set-url origin \
  "https://${GH_USER}:${GH_TOKEN}@github.com/newspapersystems/indexservice.git"
```

## Test the Fetch

```bash
runuser -u git -- \
  git --git-dir="$REPO_PATH" fetch --prune origin
```

Then remove the shell variables:

```bash
unset GH_TOKEN GH_USER
```

A successful fetch may display many deleted branches or pull-request references. That is normal when `--prune` removes references that no longer exist on GitHub.

An authentication failure will look similar to:

```text
remote: Invalid username or token.
fatal: Authentication failed
```

Do not continue with the bulk update if authentication fails.

---

# Part 5 — Verify the Test Repository in Forgejo

In the Forgejo web interface, open:

```text
newspapersystems/indexservice
→ Settings
→ Repository
→ Mirror Settings
→ Synchronize Now
```

Then check the service log:

```bash
journalctl -u forgejo --since "5 minutes ago" --no-pager |
grep -Ei 'indexservice|SyncMirrors|authentication|fatal|error'
```

The synchronization should complete without:

```text
Invalid username or token
Authentication failed
```

Once this test succeeds, continue with the bulk update.

---

# Part 6 — Bulk-Update All GitHub Mirrors

The bulk-update script should be stored at:

```text
/root/update-forgejo-github-remotes.sh
```

## Current Script

```bash
#!/usr/bin/env bash

set -uo pipefail

REPO_ROOT="/var/lib/forgejo/data/forgejo-repositories"
GITHUB_ORG="newspapersystems"
GITHUB_USER="mrmartymac"

# Start with false. Change to true only after reviewing the dry run.
APPLY_CHANGES=false

if [[ ! -d "$REPO_ROOT/$GITHUB_ORG" ]]; then
    echo "ERROR: Repository directory not found:"
    echo "       $REPO_ROOT/$GITHUB_ORG"
    exit 1
fi

read -rsp "Enter the new GitHub token: " GITHUB_TOKEN
echo

if [[ -z "$GITHUB_TOKEN" ]]; then
    echo "ERROR: No token entered."
    exit 1
fi

timestamp="$(date +%Y%m%d-%H%M%S)"

found=0
matched=0
updated=0
skipped=0
failed=0

while IFS= read -r -d '' repo_path; do
    ((found += 1))

    repo_name="$(basename "$repo_path" .git)"

    remote_name="$(
        runuser -u git -- \
            git --git-dir="$repo_path" remote 2>/dev/null |
        head -n 1
    )"

    if [[ -z "$remote_name" ]]; then
        echo "SKIP: $repo_name — no Git remote"
        ((skipped += 1))
        continue
    fi

    old_url="$(
        runuser -u git -- \
            git --git-dir="$repo_path" \
            config --get "remote.${remote_name}.url" 2>/dev/null
    )"

    # Strip credentials before examining or displaying the URL.
    safe_url="$(
        printf '%s\n' "$old_url" |
        sed -E 's#https://[^/@]+(:[^/@]*)?@#https://#'
    )"

    case "$safe_url" in
        "https://github.com/${GITHUB_ORG}/"*)
            ;;
        *)
            echo "SKIP: $repo_name — not a ${GITHUB_ORG} GitHub remote"
            ((skipped += 1))
            continue
            ;;
    esac

    ((matched += 1))

    github_path="${safe_url#https://github.com/}"
    github_path="${github_path%/}"

    new_url="https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/${github_path}"

    echo "MATCH: $GITHUB_ORG/$repo_name"

    if [[ "$APPLY_CHANGES" != "true" ]]; then
        echo "       DRY RUN — would update remote '$remote_name'"
        continue
    fi

    backup="${repo_path}/config.before-token-update-${timestamp}"

    if ! cp -a "${repo_path}/config" "$backup"; then
        echo "       ERROR — could not back up config"
        ((failed += 1))
        continue
    fi

    if runuser -u git -- \
        git --git-dir="$repo_path" \
        remote set-url "$remote_name" "$new_url"
    then
        echo "       Updated"
        ((updated += 1))
    else
        echo "       ERROR — update failed"
        cp -a "$backup" "${repo_path}/config"
        ((failed += 1))
    fi

done < <(
    find "$REPO_ROOT/$GITHUB_ORG" \
        -mindepth 1 \
        -maxdepth 1 \
        -type d \
        -name '*.git' \
        -print0
)

unset GITHUB_TOKEN

echo
echo "Repositories inspected: $found"
echo "GitHub mirrors matched:  $matched"
echo "Repositories updated:   $updated"
echo "Repositories skipped:   $skipped"
echo "Failures:               $failed"

if [[ "$APPLY_CHANGES" != "true" ]]; then
    echo
    echo "No changes were made because APPLY_CHANGES=false."
fi

if (( failed > 0 )); then
    exit 1
fi
```

Protect the script:

```bash
chmod 700 /root/update-forgejo-github-remotes.sh
```

---

# Part 7 — Run the Dry Run

Edit the script:

```bash
nano /root/update-forgejo-github-remotes.sh
```

Confirm that it contains:

```bash
APPLY_CHANGES=false
```

Run it:

```bash
/root/update-forgejo-github-remotes.sh
```

Paste the new GitHub token when prompted.

Review the final totals:

```text
Repositories inspected: <number>
GitHub mirrors matched:  <number>
Repositories updated:   0
Repositories skipped:   <number>
Failures:               0

No changes were made because APPLY_CHANGES=false.
```

The number of matched repositories should be close to the expected number of GitHub mirrors.

At the time this guide was created, approximately 179 repositories were expected.

Do not proceed if:

- The matched count is unexpectedly low.
- The script reports failures.
- The script matches repositories outside the intended GitHub organization.
- The repository root is incorrect.

---

# Part 8 — Apply the Token Update

Edit the script:

```bash
nano /root/update-forgejo-github-remotes.sh
```

Change:

```bash
APPLY_CHANGES=false
```

to:

```bash
APPLY_CHANGES=true
```

Run the script:

```bash
/root/update-forgejo-github-remotes.sh
```

Enter the new GitHub token.

The final summary should resemble:

```text
Repositories inspected: 179
GitHub mirrors matched:  179
Repositories updated:   179
Repositories skipped:   0
Failures:               0
```

Actual totals may vary as repositories are added or removed.

After the update, return the script to dry-run mode:

```bash
nano /root/update-forgejo-github-remotes.sh
```

Set:

```bash
APPLY_CHANGES=false
```

This helps prevent an accidental future bulk change.

---

# Part 9 — Verify Forgejo Synchronization

Allow Forgejo to process the mirrors normally.

Monitor live logs:

```bash
journalctl -u forgejo -f |
grep -Ei 'SyncMirrors|authentication|invalid username|fatal|error'
```

Review the previous hour:

```bash
journalctl -u forgejo --since "1 hour ago" --no-pager |
grep -Ei 'SyncMirrors|authentication|invalid username|fatal|error'
```

A failed token update typically produces:

```text
remote: Invalid username or token.
Password authentication is not supported for Git operations.
fatal: Authentication failed
```

A successful update should eliminate those messages.

Also manually test a few repositories in the Forgejo web interface:

```text
Repository
→ Settings
→ Repository
→ Mirror Settings
→ Synchronize Now
```

Recommended test repositories:

```text
newspapersystems/indexservice
newspapersystems/cas
```

---

# Part 10 — Revoke the Old GitHub Token

Only after several repositories have synchronized successfully:

1. Return to GitHub token settings.
2. Identify the old token.
3. Revoke or delete it.
4. Confirm that Forgejo continues to synchronize using the new token.

---

# Part 11 — Retain and Later Remove Backups

The script creates a backup of each repository configuration:

```text
config.before-token-update-YYYYMMDD-HHMMSS
```

Keep these files until the mirrors have synchronized successfully for several days.

To list them:

```bash
find /var/lib/forgejo/data/forgejo-repositories/newspapersystems \
  -type f \
  -name 'config.before-token-update-*' \
  -print
```

To remove backups older than 30 days:

```bash
find /var/lib/forgejo/data/forgejo-repositories/newspapersystems \
  -type f \
  -name 'config.before-token-update-*' \
  -mtime +30 \
  -delete
```

Run the listing command first and review the results before using `-delete`.

---

# Rollback Procedure

If a repository fails after the update, locate its most recent backup:

```bash
ls -lt \
  /var/lib/forgejo/data/forgejo-repositories/newspapersystems/REPOSITORY.git/config.before-token-update-*
```

Restore the desired backup:

```bash
cp -a \
  /var/lib/forgejo/data/forgejo-repositories/newspapersystems/REPOSITORY.git/config.before-token-update-YYYYMMDD-HHMMSS \
  /var/lib/forgejo/data/forgejo-repositories/newspapersystems/REPOSITORY.git/config
```

Restore ownership if necessary:

```bash
chown git:git \
  /var/lib/forgejo/data/forgejo-repositories/newspapersystems/REPOSITORY.git/config
```

Test the repository:

```bash
runuser -u git -- \
  git --git-dir="/var/lib/forgejo/data/forgejo-repositories/newspapersystems/REPOSITORY.git" \
  fetch --prune origin
```

---

# Troubleshooting

## Invalid Username or Token

```text
remote: Invalid username or token.
fatal: Authentication failed
```

Check:

- The token was copied completely.
- The token has not expired.
- The token has access to the repository.
- The username is `mrmartymac`.
- The token is authorized for the `newspapersystems` organization.
- The repository is included in the fine-grained token.

## Repository Not Found

```text
remote: Repository not found.
```

Check:

- The repository still exists on GitHub.
- The repository has not been renamed or transferred.
- The token owner has access.
- The repository was included in the token’s repository selection.

## Unknown Linux User

```text
sudo: unknown user forgejo
```

The Forgejo process on this container runs as:

```text
git
```

Use:

```bash
runuser -u git -- <command>
```

Do not use:

```bash
sudo -u forgejo
```

## Large Number of Deleted References

A successful command such as:

```bash
git fetch --prune
```

may show many lines like:

```text
- [deleted] (none) -> branch-name
- [deleted] (none) -> refs/pull/123/head
```

This is expected. Git is pruning references that no longer exist on GitHub.

## Confirm the Forgejo Service Account

```bash
systemctl show forgejo -p User -p Group
```

Expected result:

```text
User=git
Group=git
```

---

# Annual Checklist

```text
[ ] Create new GitHub fine-grained token
[ ] Do not revoke old token yet
[ ] Enter the Forgejo CT
[ ] Test token with git ls-remote
[ ] Update and fetch one test mirror
[ ] Synchronize test repository through Forgejo
[ ] Confirm no authentication errors
[ ] Run bulk script with APPLY_CHANGES=false
[ ] Review repository counts
[ ] Run bulk script with APPLY_CHANGES=true
[ ] Return APPLY_CHANGES to false
[ ] Monitor Forgejo logs
[ ] Manually test several mirrors
[ ] Revoke old GitHub token
[ ] Retain backup configuration files temporarily
[ ] Delete backups older than 30 days after stability is confirmed
[ ] Record the new token expiration date
```

---

# Recommended Recordkeeping

Record the following in the appropriate secure administrative documentation:

```text
Token creation date:
Token expiration date:
GitHub account:
GitHub organization:
Forgejo server:
Forgejo version:
Number of mirrors updated:
Date old token was revoked:
Administrator performing rotation:
```

Do not record the token value itself.

---

## Document History

```text
Created: August 6, 2026
Environment: Forgejo 15.0.2 on Proxmox LXC
Mirror direction: GitHub to Forgejo
GitHub organization: newspapersystems
Forgejo service account: git
```
