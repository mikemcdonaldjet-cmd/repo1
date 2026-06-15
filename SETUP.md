# Weighted Items — Daily RAID Analysis: Setup Guide

## What this does

Every weekday at **5:30pm UK time**, a GitHub Actions workflow wakes up, runs Claude, and:

1. Reads your Google Calendar for today's meetings and any linked Google Docs
2. Scans five Slack channels for today's messages
3. Checks the `WI` Jira project for duplicates
4. **Publishes a structured proposal table to the GitHub Actions Summary page**

You review proposals there and approve via a new Claude Code session. No Jira tickets are ever created without your explicit go-ahead.

---

## Where to find the daily summary

Go to: **Actions → Mikes-Weighted-Items-RAID-Analysis → [latest run] → Summary tab**

Bookmark this URL for quick access:
`https://github.com/mikemcdonaldjet-cmd/repo1/actions/workflows/weighted-items-daily-raid.yml`

---

## One-time setup

### 1. GitHub Secrets

Go to **Settings → Secrets and variables → Actions → New repository secret** and add these:

| Secret name | How to get it |
|---|---|
| `ANTHROPIC_API_KEY` | https://console.anthropic.com → API Keys |
| `SLACK_BOT_TOKEN` | See "Slack setup" below |
| `GOOGLE_CLIENT_ID` | See "Google setup" below |
| `GOOGLE_CLIENT_SECRET` | See "Google setup" below |
| `GOOGLE_REFRESH_TOKEN` | See "Google setup" below |
| `ATLASSIAN_SITE_URL` | Your Jira site URL, e.g. `https://justeattakeaway.atlassian.net` |
| `ATLASSIAN_API_TOKEN` | https://id.atlassian.com → Security → API tokens → Create |

---

### 2. Slack setup (read-only token for scanning channels)

You need a Slack token that can **read** the five channels. The simplest way is a **User OAuth Token** from a Slack app with read-only scopes — no posting permissions required.

OAuth scopes needed:
- `channels:history` / `groups:history` — read channel messages
- `channels:read` / `groups:read` — list channels
- `users:read` — look up user names

Steps:
1. Go to https://api.slack.com/apps → **Create New App → From scratch**
2. Name it `Weighted Items Reader`, pick the JET workspace
3. Go to **OAuth & Permissions** → add the scopes above
4. **Install to Workspace** → copy the **Bot User OAuth Token** (`xoxb-...`)
5. Invite the app to the five scanned channels: `/invite @weighted-items-reader`
6. Add the token as `SLACK_BOT_TOKEN` in GitHub Secrets

---

### 3. Google setup (Calendar + Drive)

You need OAuth 2.0 credentials so Claude can read your calendar and docs headlessly.

Steps:
1. Go to https://console.cloud.google.com → create or pick a project
2. Enable **Google Calendar API** and **Google Drive API**
3. Go to **APIs & Services → Credentials → Create OAuth Client ID → Desktop app**
4. Download the JSON — note `client_id` and `client_secret`
5. Run this locally once to get a refresh token:

```bash
npx @modelcontextprotocol/server-google-drive --auth
# Follow the browser prompt — it prints a refresh token
```

6. Add `client_id`, `client_secret`, and `refresh_token` as GitHub Secrets.

---

### 4. Verify MCP package names

The workflow uses these npm packages. If a name has changed, update the workflow file:

| MCP server | Package |
|---|---|
| Slack | `@modelcontextprotocol/server-slack` |
| Google Calendar | `@modelcontextprotocol/server-google-calendar` |
| Google Drive | `@modelcontextprotocol/server-google-drive` |
| Atlassian/Jira | `@atlassian/mcp-atlassian` |

Current package list: https://github.com/modelcontextprotocol/servers

---

### 5. Timezone adjustment

The cron fires at `30 16 * * 1-5` (4:30pm UTC = **5:30pm BST**, UK summer).

During **winter (GMT, October–March)**, change it to `30 17 * * 1-5` so it fires at 5:30pm GMT.

---

## Running manually

Go to **Actions → Mikes-Weighted-Items-RAID-Analysis → Run workflow** to trigger a test run at any time.

---

## Approving actions

1. Open the **Summary tab** of the latest Actions run
2. Review the proposed creates and updates
3. Start a new Claude Code session and paste: _"Approve items 1, 3, 5 from today's Weighted Items update"_ (or _"Approve all"_)
4. Claude will create/update the Jira tickets in the `WI` project
