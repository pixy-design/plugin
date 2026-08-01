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
3. **`NEED_KEY`** → call `pixy_pair` (context: folder, git remote, dev
   URL), show the link — one line: "Connect this project to your Pixy
   account: {link}" — poll `pixy_pair` (code + secret) every ~5s until
   approved, then re-run the script with `--key <the key>`. Link dead
   after 10 min → restart pairing once, then ask for the key
   (https://pixydesignapp.com/@my).
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
   session start, offer to connect right away instead. `BAD_KEY` → the key
   is stale — pair again (step 3).
