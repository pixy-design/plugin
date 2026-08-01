---
description: Wire this project for Pixy — one script does the mechanics, you wire the tag
---

This file is the COMPLETE setup instruction. Do not load any skill, do not
read plugin or skill files, never search `~/.claude` or the plugin cache,
never go looking for a key anywhere (the script below does key detection,
and step 3 says where a key may legitimately come from). The
mechanics are a script; your own work is ONE discovery batch and (at most)
ONE edit. Target: **under a minute of your work** plus the user's pairing
click.

1. **Discovery — ONE parallel batch, then stop**: read `package.json` (or
   launch config / run script) for the dev command and URL/port; Grep the
   source for an existing `pixy.mjs` tag; Glob for what produces the page
   head (`index.html`, root layout, base template).

   That batch is all the looking you get. **If it doesn't hand you a dev
   command and a URL, do not search again** — no second grep, no walking
   the tree, no reading configs to infer a port. Ask the user one
   question ("what starts this project, and what URL does it serve on?"),
   say what you did find, and wait. A project whose layout you can't read
   in one batch is a project only the user can explain, and guessing at it
   is how the wrong file ends up wired.

   Reading the grep: only a literal `<script … pixy.mjs?k=…>` in a page or
   template is an existing tag. Code that *serves*, *generates* or
   *documents* that string is not — a match inside a route handler, a
   string built at runtime, or a copy-paste snippet means you are in
   something that ships Pixy, not something wired for it.
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

   **Only ever pass `--key` with a key the user typed into chat.** Never
   one you found while looking around — not from a config file, not from
   an env dump, not from a page or a fixture. A `px_…` sitting in the
   project's own files belongs to whatever that project ships; it is not
   this project's wiring, and handing it to the script buys a green
   MODULE_OK for the wrong account. If you have no key, you have no key:
   let the script pair (step 3), which is the whole point of it.
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
