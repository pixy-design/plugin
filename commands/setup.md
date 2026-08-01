---
description: Wire this project for Pixy — one script does the mechanics, you wire the tag
---

This file is the COMPLETE setup instruction. Do not load any skill, do not
read plugin or skill files, never search `~/.claude` or the plugin cache,
never hunt for env vars (the script below does key detection). The
mechanics are a script; your own work is ONE discovery batch and (at most)
ONE edit. Target: **under a minute of your work** plus the user's pairing
click.

1. **Discovery — ONE parallel batch, nothing more**: read `package.json`
   (or launch config / run script) for the dev command and URL/port; Grep
   the source for an existing `pixy.mjs` tag; Glob for what produces the
   page head (`index.html`, root layout, base template).
2. **Run the setup script** from the project root (fills in what step 1
   found; omit flags you don't know; `--external` = user runs their own
   server; `--tag` = the file already carrying a tag, or the file you'll
   put it in):

   ```bash
   curl -fsSL https://mcp.pixydesignapp.com/setup | bash -s -- \
     --dev "npm run dev" --url "http://localhost:3000" --tag "src/app/layout.tsx"
   ```

   It finds the key (env → project settings), writes `.claude/settings.json`
   (merged: key, allowlist, acceptEdits), `.claude/pixy-connect`,
   `.claude/pixy.json`, and verifies both ends. Its output is a fixed
   summary with a `NEXT:` line — follow that line literally.
3. **`PAIRING <link>`** → the script started the pairing itself; you do not
   call `pixy_pair` and you do not poll. Show the user the one line it
   printed — "Connect this project to your Pixy account: {link}" — then
   re-run the same command plus `--wait`. That run blocks on their
   approval and finishes setup in one go (give the Bash call a 600000ms
   timeout). `PAIR_PENDING` → they haven't clicked yet: one line nudging
   them, re-run with `--wait`. `PAIR_UNAVAILABLE` → ask for a key from
   https://pixydesignapp.com/@my and re-run with `--key <the key>`. A key
   that has gone stale needs nothing from you either — the script clears
   it and re-pairs in the same run.
4. **`TAG_MISSING`** → your one edit: insert the tag (the script printed
   the exact element) into the page head, dev-gated by the project's own
   idiom (`NODE_ENV === "development"`, a dev template block, a server dev
   flag); no dev/prod split → plain tag plus one line to the user that it
   ships until removed. If step 1 found an existing tag, update its key
   instead — never add a second. Restart the dev server if hot reload
   skips template/config files; a user-managed server gets one line:
   "restart your dev server to pick up the tag". Re-run the script (same
   flags plus `--tag <file>`) to re-verify.
5. **`TAG_OK`** → done: tell the user to restart Claude Code once (arms
   the key and permissions), then `/pixy`. If the key was already armed at
   session start, offer to connect right away instead.

Every run ends in one of exactly two places — `TAG_MISSING` (step 4) or
`TAG_OK` (step 5). Anything else is the script still waiting on the user's
click, and the answer is always to show the link and re-run with `--wait`.
