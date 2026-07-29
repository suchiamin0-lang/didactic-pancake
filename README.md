# Claude + Gmail (multiple accounts)

This repo connects **Claude** to **Gmail** so you can read and manage your mail
by just talking to Claude — e.g. *"What did I get from the bank this week?"* or
*"Search all my accounts for the flight confirmation and summarize it."*

It's set up to scan **more than one Gmail account at once**. Each account runs as
its own **MCP server** instance, so you can ask Claude to look across your
personal and work inboxes in the same conversation.

It uses the open-source [`@gongrzhe/server-gmail-autoauth-mcp`](https://github.com/GongRzhe/Gmail-MCP-Server)
server (an unofficial, third-party tool — not affiliated with Google), wired up
via [`.mcp.json`](./.mcp.json). You don't need to install or build anything by
hand; the configuration here runs it automatically with `npx`.

## What you get

Two Gmail connectors are pre-configured out of the box:

- `gmail-personal`
- `gmail-work`

Once connected, you can ask Claude things like:

- "Search both my inboxes for emails from Stripe in the last 30 days."
- "Summarize today's unread mail in my work account."
- "Find the Amazon order confirmation in my personal account."
- "Draft a reply to the latest email from my landlord."

You can rename these or add more accounts — see
[Adding or renaming accounts](#adding-or-renaming-accounts) below.

## How multiple accounts work

The Gmail server authenticates **one account per running instance**. To scan
several accounts, this repo runs the same server multiple times — once per
account — with each instance pointed at its **own credentials file**:

- All accounts share **one** Google Cloud OAuth app (`GMAIL_OAUTH_PATH`).
- Each account gets its **own** login token file (`GMAIL_*_CREDENTIALS_PATH`).

## One-time setup

### 1. Create a Google Cloud OAuth app (shared by all accounts)

1. Go to <https://console.cloud.google.com> and create (or pick) a project.
2. Enable the **Gmail API** for that project.
3. Under **APIs & Services → Credentials**, create an **OAuth 2.0 Client ID**
   of type **Desktop app** and download the JSON (commonly named
   `gcp-oauth.keys.json`).
4. Save it somewhere safe outside this repo.

### 2. Configure paths (kept out of git)

```bash
cp .env.example .env
```

Open `.env` and set:

- `GMAIL_OAUTH_PATH` — path to the `gcp-oauth.keys.json` from step 1.
- `GMAIL_PERSONAL_CREDENTIALS_PATH` — where to store the personal account token.
- `GMAIL_WORK_CREDENTIALS_PATH` — where to store the work account token.

Give each account a **distinct** credentials path so they don't overwrite each
other. The `.env` file and any `credentials*.json` are git-ignored, so your
secrets stay private and are never committed.

> **Keep these secret.** The OAuth keys and per-account tokens are like
> passwords for your Gmail accounts.

### 3. Authenticate each account

Run the server's one-time auth flow **once per account**, pointing at that
account's credentials path. A browser window opens for you to grant access, and
the token is written to the path you specify:

```bash
# Personal account
GMAIL_OAUTH_PATH="$GMAIL_OAUTH_PATH" \
GMAIL_CREDENTIALS_PATH="$GMAIL_PERSONAL_CREDENTIALS_PATH" \
npx -y @gongrzhe/server-gmail-autoauth-mcp auth

# Work account (sign in with your OTHER Google account when the browser opens)
GMAIL_OAUTH_PATH="$GMAIL_OAUTH_PATH" \
GMAIL_CREDENTIALS_PATH="$GMAIL_WORK_CREDENTIALS_PATH" \
npx -y @gongrzhe/server-gmail-autoauth-mcp auth
```

Make sure the variables are loaded first, e.g. with
[`direnv`](https://direnv.net) or `set -a; source .env; set +a`.

### 4. Start Claude Code in this folder

Claude Code automatically picks up the `.mcp.json` in this repo and starts both
Gmail servers. (You may be prompted once to approve each new MCP server.)
Verify it works by asking:

> Search my personal and work Gmail for anything from the last 24 hours.

If it returns results from both, you're connected. 🎉

## Adding or renaming accounts

Each account is just an entry under `mcpServers` in [`.mcp.json`](./.mcp.json).
To add a third account, copy one of the blocks, give it a new name, point it at a
new credentials path, add that variable to `.env`, and run the `auth` step for
it:

```json
{
  "mcpServers": {
    "gmail-personal": {
      "command": "npx",
      "args": ["-y", "@gongrzhe/server-gmail-autoauth-mcp"],
      "env": {
        "GMAIL_OAUTH_PATH": "${GMAIL_OAUTH_PATH}",
        "GMAIL_CREDENTIALS_PATH": "${GMAIL_PERSONAL_CREDENTIALS_PATH}"
      }
    }
  }
}
```

## Using it in other Claude clients

The same servers work in Claude Desktop, Cursor, VS Code, and others. Point your
client's MCP settings at the command shown in [`.mcp.json`](./.mcp.json).

## Requirements

- [Node.js](https://nodejs.org) 22 or newer (so `npx` can run the servers).

## Troubleshooting

- **"Server failed to start" / no tools:** make sure `GMAIL_OAUTH_PATH` and each
  `GMAIL_*_CREDENTIALS_PATH` are set in the same terminal/environment where
  Claude is running.
- **One account works but another doesn't:** confirm each account has its own
  credentials path and that you ran the `auth` step for that account.
- **"Not authorized" or expired token:** re-run the `auth` step (step 3) for the
  affected account to refresh its token.
- **Need details on the server itself:** see the upstream project,
  <https://github.com/GongRzhe/Gmail-MCP-Server>.
