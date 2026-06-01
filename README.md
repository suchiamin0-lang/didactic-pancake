# Claude + Wanderlog

This repo connects **Claude** to **[Wanderlog](https://wanderlog.com)** so you can
plan trips by just talking to Claude — e.g. *"Create a 7-day Italy itinerary covering
Rome, Florence, and Venice"* — and it fills in places, notes, hotels, and checklists
in your Wanderlog account for you.

It does this using the open-source [`wanderlog-mcp`](https://github.com/shaikhspeare/wanderlog-mcp)
server (an unofficial, third-party tool — not affiliated with Wanderlog), wired up as
an **MCP server**. You don't need to install or build anything by hand; the
configuration here runs it automatically with `npx`.

## What you get

Once connected, you can ask Claude things like:

- "What trips do I have in Wanderlog?"
- "Create a 14-day Japan trip from May 1, covering Tokyo, Hakone, Kyoto, Nara, and Osaka."
- "Add a sushi restaurant near our Kyoto hotel on day 4."
- "Add a hotel in Osaka with check-in May 12 and check-out May 14."
- "Make a pre-trip packing checklist."

Behind the scenes Claude uses tools like `wanderlog_list_trips`, `wanderlog_create_trip`,
`wanderlog_add_place`, `wanderlog_add_hotel`, `wanderlog_add_note`, and more.

## One-time setup

The only thing you have to provide is your Wanderlog login cookie, so the server can
act on your behalf.

### 1. Get your Wanderlog cookie

1. Log in at <https://wanderlog.com> in your browser.
2. Open DevTools (press **F12**), then:
   - Chrome / Edge: **Application** tab
   - Firefox: **Storage** tab
3. In the sidebar, expand **Cookies → https://wanderlog.com**.
4. Click the cookie named **`connect.sid`** and copy its full **Value**
   (it starts with `s%3A` and is ~100 characters long).

### 2. Save it locally (kept out of git)

```bash
cp .env.example .env
```

Open `.env` and paste your cookie value after `WANDERLOG_COOKIE=`.
The `.env` file is git-ignored, so your cookie stays private and is never committed.

> **Keep this secret.** The cookie is like a password for your Wanderlog account.
> It typically lasts about a year, but will stop working if you log out, change your
> password, or Wanderlog revokes the session.

### 3. Make sure the variable is available

The MCP config reads the cookie from the `WANDERLOG_COOKIE` environment variable.
Either load your `.env` file (e.g. with [`direnv`](https://direnv.net) or
`set -a; source .env; set +a`) or export it directly:

```bash
export WANDERLOG_COOKIE='s%3A...your value...'
```

### 4. Start Claude Code in this folder

Claude Code automatically picks up the `.mcp.json` in this repo and starts the
`wanderlog` server. (You may be prompted once to approve the new MCP server.)
Verify it works by asking:

> What trips do I have in Wanderlog?

If it lists your trips, you're connected. 🎉

## Using it in other Claude clients

The same server works in Claude Desktop, Cursor, VS Code, and others. The config in
[`.mcp.json`](./.mcp.json) shows the command — point your client's MCP settings at:

```json
{
  "mcpServers": {
    "wanderlog": {
      "command": "npx",
      "args": ["-y", "wanderlog-mcp"],
      "env": { "WANDERLOG_COOKIE": "s%3A...your value..." }
    }
  }
}
```

## Requirements

- [Node.js](https://nodejs.org) 22 or newer (so `npx` can run the server).

## Troubleshooting

- **"Server failed to start" / no tools:** make sure `WANDERLOG_COOKIE` is set in the
  same terminal/environment where Claude is running.
- **"Not authorized" or empty results:** your cookie may have expired — repeat step 1
  to grab a fresh `connect.sid`.
- **Need details on the server itself:** see the upstream project,
  <https://github.com/shaikhspeare/wanderlog-mcp>.
