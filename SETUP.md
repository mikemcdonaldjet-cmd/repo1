# Weighted Items — Daily RAID Analysis: Setup Guide

## What this does

Every weekday at **5:30pm UK time**, a GitHub Actions workflow wakes up, runs Claude, and:

1. Reads your Google Calendar for today's meetings and any linked Google Docs
2. Scans five Slack channels for today's messages
3. Checks the `WI` Jira project for duplicates
4. **DMs you on Slack** with a structured list of proposed ticket actions

You then reply to the Slack DM to approve (or skip) each action. No Jira tickets are ever created without your explicit go-ahead.

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

### 2. Slack setup

You need a Slack app with a **Bot Token** that has these OAuth scopes:

- `channels:history` / `groups:history` — read channel messages
- `channels:read` / `groups:read` — list channels
- `users:read` — look up users
- `chat:write` — DM you with the daily summary
- `im:write` — open DM channels

Steps:
1. Go to https://api.slack.com/apps → **Create New App** → From scratch
2. Name it `Weighted Items Bot`, pick your JET workspace
3. Go to **OAuth & Permissions** → add the scopes above
4. **Install to Workspace** → copy the **Bot User OAuth Token** (`xoxb-...`)
5. Invite the bot to each of the five scanned channels: `/invite @weighted-items-bot`

---

### 3. Google setup (Calendar + Drive)

You need OAuth 2.0 credentials with offline access to your Google account.

Steps:
1. Go to https://console.cloud.google.com → Create a project (or use an existing one)
2. Enable **Google Calendar API** and **Google Drive API**
3. Go to **APIs & Services → Credentials → Create OAuth Client ID** → Desktop app
4. Download the JSON, note `client_id` and `client_secret`
5. Run this one-time script locally to get a refresh token:

```bash
npx @modelcontextprotocol/server-google-calendar --auth
# Follow the browser prompt, paste the code back — it prints a refresh token
```

6. Add `client_id`, `client_secret`, and `refresh_token` as GitHub secrets.

---

### 4. Verify the MCP package names

The workflow uses these npm packages as MCP servers. If a package name has changed, update `.github/workflows/weighted-items-daily-raid.yml`:

| MCP server | Package |
|---|---|
| Slack | `@modelcontextprotocol/server-slack` |
| Google Calendar | `@modelcontextprotocol/server-google-calendar` |
| Google Drive | `@modelcontextprotocol/server-google-drive` |
| Atlassian/Jira | `@atlassian/mcp-atlassian` |

Check current package names at https://github.com/modelcontextprotocol/servers.

---

### 5. Timezone adjustment

The cron is set to `30 16 * * 1-5` (4:30pm UTC = 5:30pm BST, UK summer time).

During **winter (GMT)**, change it to `30 17 * * 1-5` in the workflow file, or the run will fire at 4:30pm instead.

---

## Running manually

You can trigger the workflow anytime from **Actions → Mikes-Weighted-Items-RAID-Analysis → Run workflow**.

---

## Approving actions

After each daily run you'll receive a Slack DM from the bot. Reply:
- **✅ All** — approve everything proposed
- **✅ 1, 3, 5** — approve specific items by number
- **❌** — skip everything today

Then start a new Claude Code session and paste the approved items — Claude will create/update the Jira tickets in the `WI` project.
