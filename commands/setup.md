---
description: Wire this project for Pixy — pairing, permissions, script tag, verification
---

This file is the COMPLETE setup instruction. Do not load any skill, do not
read plugin or skill files, never search `~/.claude` or the plugin cache.
Setup is copy-paste with two variables — the key and the dev command — plus
one real edit (the script tag). Target: **under two minutes of your own
work**; the pairing approval is the user's time and you work through it.

1. **Discovery — ONE parallel batch, nothing more**: read `package.json`
   (or launch config / run script) for the dev command and port; Grep the
   source for an existing `pixy.mjs` tag; Glob for what produces the page
   head (`index.html`, root layout, base template).
2. **Key**: session env `PIXY_KEY` → `env.PIXY_KEY` in
   `.claude/settings.json` → a key in the user's message. None → call
   `pixy_pair` (context: folder, git remote, dev URL), show the link — one
   line: "Connect this project to your Pixy account: {link}" — then do
   steps 3–5 WHILE polling `pixy_pair` (code + secret) every ~5s. Link dead
   after 10 min → restart pairing once, then ask for the key
   (https://pixydesignapp.com/@my).
3. **Settings** — merge into `.claude/settings.json`, preserve what's
   there; `<dev>` is this project's real dev-server command; append the
   project's other routine commands (its build/check/test):

   ```json
   {
   	"env": { "PIXY_KEY": "<the key>" },
   	"permissions": {
   		"allow": [
   			"mcp__plugin_pixy_pixy__*",
   			"mcp__Claude_Browser__*",
   			"Bash(<dev>:*)",
   			"Bash(bash .claude/pixy-connect:*)",
   			"Bash(ls:*)", "Bash(cat:*)", "Bash(head:*)", "Bash(tail:*)",
   			"Bash(grep:*)", "Bash(rg:*)", "Bash(find:*)", "Bash(wc:*)",
   			"Bash(curl -s:*)",
   			"Bash(git status:*)", "Bash(git diff:*)", "Bash(git log:*)"
   		],
   		"defaultMode": "acceptEdits"
   	}
   }
   ```

4. **Preflight script** — write `.claude/pixy-connect` EXACTLY as below
   (invoked via `bash`, no chmod needed):

   ```bash
   #!/bin/bash
   # Written by /pixy setup — zero-thought preflight for /pixy connect.
   cd "$(dirname "$0")/.."
   [ -f .claude/pixy.json ] || { echo "NO_STATE — run /pixy setup"; exit 0; }
   cat .claude/pixy.json
   URL=$(python3 -c "import json;print(json.load(open('.claude/pixy.json'))['url'])")
   CODE=$(curl -sI -m 5 -o /dev/null -w "%{http_code}" "$URL" 2>/dev/null)
   case "$CODE" in
   	2*|3*) echo "SERVER_UP $URL" ;;
   	*)     echo "SERVER_DOWN $URL" ;;
   esac
   ```

5. **Script tag — the only thinking step**: step 1 found a `pixy.mjs` tag →
   update its key if stale, record its file, never add a second. Otherwise
   insert into the head, dev-gated by the project's own idiom
   (`NODE_ENV === "development"`, a dev template block, a server dev flag);
   no dev/prod split (plain static site) → plain tag plus one line to the
   user that it ships until removed:

   ```html
   <script type="module" src="{origin}/pixy.mjs?k={key}"></script>
   ```

   `{origin}`: `https://pixydesignapp.com` for `px_` keys,
   `https://sandbox.pixydesignapp.com` for `pxs_` keys.
6. **Verify — exactly two fetches, fix what fails**: the dev URL serves
   HTML containing the tag (start/restart the dev server if hot reload
   skipped the template edit; user-managed server → record
   `server: "external"` and say one line: "restart your dev server to pick
   up the tag"); `{origin}/pixy.mjs?k={key}` serves the real module (a stub
   mentioning "personal key" = bad or stale key).
7. **Finish**: write `.claude/pixy.json` —
   `{ "url": …, "server": …, "tag": …, "origin": … }` (`server`: the dev
   command, or `"external"`; `tag`: the file holding the tag, or
   `"manual"`). One closing message: setup done, restart Claude Code once
   to arm the key and permissions, then `/pixy`. If the key was already
   armed at session start, offer to connect right away instead.
