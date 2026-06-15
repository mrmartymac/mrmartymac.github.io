---
title: Automating Forgejo mirror when new Github repo is created
date: 2026-05-29 09:42:09 -400
categories: [Documentation, Forgejo]
tags: [git, github, forgejo, proxmox] # TAG names should always be lowercase
author: mm
---

## Creating the intial mirror of the repos

The script `mirror_github.sh` resides in `/home/git` on the Forgejo instance on ProxMox1. The script is configured to create mirrors of repos in the newspapersystem Github account.

```bash
#!/bin/bash

# --- CONFIGURATION ---
GITHUB_ORG="newspapersystems"
GITHUB_TOKEN="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" # Required to bypass GitHub API rate limits
GITHUB_USERNAME="mrmartymac"    # <--- Put your real GitHub username here

FORGEJO_URL="http://localhost:3000"              # Update to your internal container port/URL if different
FORGEJO_TOKEN="074c75955d340a49e2393e0f3403d6de0027ea1f"        # The token you generated in Step 1
FORGEJO_OWNER="newspapersystems"     # The target user/org on Forgejo where mirrors will live
# ---------------------

LIST_FILE="repos.txt"
# ---------------------

# Verify that the repository list file exists
if [ ! -f "$LIST_FILE" ]; then
    echo "Error: Target file '$LIST_FILE' not found. Please create it first."
    exit 1
fi

echo "Reading repository targets from $LIST_FILE..."

# Read the file line by line safely
while IFS= read -r REPO || [ -n "$REPO" ]; do
    # Trim whitespace
    REPO=$(echo "$REPO" | xargs)

    # Skip empty lines or lines starting with a comment '#'
    if [ -z "$REPO" ] || [[ "$REPO" =~ ^# ]]; then
        continue
    fi

    echo "----------------------------------------------------"
    echo "Configuring pull mirror for: $REPO"
    echo "----------------------------------------------------"

    # Clean target URL
    REMOTE_URL="https://github.com/${GITHUB_ORG}/${REPO}.git"

    # Construct the JSON payload using jq
    JSON_PAYLOAD=$(jq -n \
      --arg clone_addr "$REMOTE_URL" \
      --arg auth_user "$GITHUB_USERNAME" \
      --arg auth_pass "$GITHUB_TOKEN" \
      --arg r_name "$REPO" \
      --arg r_owner "$FORGEJO_OWNER" \
      '{
        clone_addr: $clone_addr,
        auth_username: $auth_user,
        auth_password: $auth_pass,
        mirror: true,
        mirror_interval: "8h0m0s",
        repo_name: $r_name,
        repo_owner: $r_owner,
        service: "git",
        private: true,
        lfs: true
      }')

    # Send the request to Forgejo's local API and capture HTTP status code
    RESPONSE_BLOCK=$(curl -s -w "\nHTTP_STATUS:%{http_code}" -X POST "${FORGEJO_URL}/api/v1/repos/migrate" \
      -H "accept: application/json" \
      -H "Authorization: token ${FORGEJO_TOKEN}" \
      -H "Content-Type: application/json" \
     -d "$JSON_PAYLOAD")

    # Separate response body from the HTTP Status Code
    RESPONSE=$(echo "$RESPONSE_BLOCK" | sed '/HTTP_STATUS:/d')
    HTTP_STATUS=$(echo "$RESPONSE_BLOCK" | grep "HTTP_STATUS:" | cut -d':' -f2)

    # Evaluate the status
    if [ "$HTTP_STATUS" -eq 201 ]; then
        echo "Success: Mirror established for $REPO."
    elif echo "$RESPONSE" | grep -q "already exists"; then
        echo "Notice: Repository $REPO already exists on Forgejo. Skipping."
    else
        echo "Error creating mirror for $REPO. HTTP Status: $HTTP_STATUS"
        echo "Response was: $RESPONSE"
    fi

done < "$LIST_FILE"

echo "----------------------------------------------------"
echo "All repositories inside $LIST_FILE have been processed!"
```

The script relies on a local file called `repos.txt` which has the names of the repos it needs to mirror.

---

## Cron task to mirror new repos

The script below can be completely automated and safe to run on a repeated `cron` schedule, we can combine our original GitHub API logic (filtering for all repository types) with the robust `service: "git"` authentication strategy we discovered.

To make it production-ready for `cron`, this automated version includes **absolute paths** (so cron knows exactly where to find `curl` and `jq`) and silences unnecessary terminal output while still logging structural changes.

## The Automated Cron Script

Create a new file in your Proxmox container (e.g., `/usr/local/bin/sync_github_mirrors.sh`):

```bash
#!/bin/bash

# --- CONFIGURATION ---
GITHUB_ORG="newspapersystems"
GITHUB_TOKEN="your_github_personal_access_token" 
GITHUB_USERNAME="your_actual_github_username"    

FORGEJO_URL="http://localhost:3000"              
FORGEJO_TOKEN="your_forgejo_access_token"        
FORGEJO_OWNER="your_forgejo_username_or_org"     
# ---------------------

# Absolute paths for cron environment stability
CURL_BIN="/usr/bin/curl"
JQ_BIN="/usr/bin/jq"

# Fetch ALL repository names (public, private, archived) from the GitHub Org
REPO_NAMES=$($CURL_BIN -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/orgs/$GITHUB_ORG/repos?type=all&per_page=100" | $JQ_BIN -r '.[].name')

# Safeguard: Exit if the GitHub API fails or returns empty
if [ -z "$REPO_NAMES" ] || [ "$REPO_NAMES" == "null" ]; then
    echo "$(date): Error fetching repository list from GitHub."
    exit 1
fi

# Loop through every repository found on GitHub
for REPO in $REPO_NAMES; do
    REMOTE_URL="https://github.com/${GITHUB_ORG}/${REPO}.git"

    # Construct the migration JSON payload
    JSON_PAYLOAD=$($JQ_BIN -n \
      --arg clone_addr "$REMOTE_URL" \
      --arg auth_user "$GITHUB_USERNAME" \
      --arg auth_pass "$GITHUB_TOKEN" \
      --arg r_name "$REPO" \
      --arg r_owner "$FORGEJO_OWNER" \
      '{
        clone_addr: $clone_addr,
        auth_username: $auth_user,
        auth_password: $auth_pass,
        mirror: true,
        mirror_interval: "8h0m0s",
        repo_name: $r_name,
        repo_owner: $r_owner,
        service: "git",
        private: true,
        lfs: true
      }')

    # Submit request to Forgejo API
    RESPONSE_BLOCK=$($CURL_BIN -s -w "\nHTTP_STATUS:%{http_code}" -X POST "${FORGEJO_URL}/api/v1/repos/migrate" \
      -H "accept: application/json" \
      -H "Authorization: token ${FORGEJO_TOKEN}" \
      -H "Content-Type: application/json" \
      -d "$JSON_PAYLOAD")

    RESPONSE=$(echo "$RESPONSE_BLOCK" | sed '/HTTP_STATUS:/d')
    HTTP_STATUS=$(echo "$RESPONSE_BLOCK" | grep "HTTP_STATUS:" | cut -d':' -f2)

    # Logging logic optimized for cron
    if [ "$HTTP_STATUS" -eq 201 ]; then
        echo "$(date): Success - New mirror established for $REPO."
    elif echo "$RESPONSE" | grep -q "already exists"; then
        # Silently skip existing repositories to keep logs clean
        continue
    else
        echo "$(date): Error syncing $REPO. HTTP $HTTP_STATUS - $RESPONSE"
    fi
done
```

### Permissions Setup

Make the file executable so the system can run it independently:

```bash
chmod +x /usr/local/bin/sync_github_mirrors.sh
```

## Scheduling via Cron

Open the root user's crontab inside your container:

```bash
crontab -e
```

Add a line at the very bottom of the file to schedule how often you want to look for new repositories.

* **Option A: Run every night at midnight**  

```text
0 0 * * * /usr/local/bin/sync_github_mirrors.sh >> /var/log/forgejo_mirror_sync.log 2>&1
```

* **Option B: Run once every hour**  

```text
0 * * * * /usr/local/bin/sync_github_mirrors.sh >> /var/log/forgejo_mirror_sync.log 2>&1
```

### Why this structure works perfectly for automation:

1. **Idempotent / Skip Logic:** Because we kept the `already exists` parsing condition, the script will loop through all your GitHub repos, see that 99% of them are already mirrored on Forgejo, and silently skip them without throwing an error or touching the existing data.
2. **Auto-Discovery:** The moment a teammate creates a new repository on GitHub (whether public or private), the next cron cycle will dynamically pull its name from the API and instantly register it as a pulling mirror on Forgejo.
3. **Log Trailing:** By sending the output to `/var/log/forgejo_mirror_sync.log`, you can check on its status at any time using `cat /var/log/forgejo_mirror_sync.log`. It will only log entries when a brand new repo is found or if an explicit API error occurs.