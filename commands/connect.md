---
description: Start a Pixy live session — design in your page, Claude builds on "go"
---

This file is the COMPLETE instruction for connecting AND for wiring a
project that isn't wired yet. Do not load any skill, do not read plugin or
skill files, never search `~/.claude` or the plugin cache.

**Read `.claude/pixy.json` and branch on it. That file is the only thing
you inspect before acting.** No searching the project, no reading
`package.json`, no grepping for `pixy.mjs`, no looking for a dev command or
a URL — every one of those answers is in that file or comes from the user.
There is nothing to discover.

## No file → pair

**Ask nothing. Not the URL, not the dev command, not sandbox.** The user
types the page URL on the approve page, where they already are, and it
comes back with the key.

1. Run, from the project root:

   ```bash
   curl -fsSL https://mcp.pixydesignapp.com/setup | bash -s --
   ```

2. **`PAIRING <link>`** → show the user the one line it printed, then
   re-run the same command plus `--wait` (give the Bash call a 600000ms
   timeout). That run blocks on their approval and writes
   `.claude/settings.json` and `.claude/pixy.json`. `PAIR_PENDING` → one
   line nudging them, re-run with `--wait`. `PAIR_UNAVAILABLE` → ask for a
   key from the dashboard, re-run with `--key <the key>`.
3. Then continue at **`ready: false`** below.

**Only ever pass `--key` with a key the user typed into chat.** Never one
you found while looking around — a `px_…` in the project's own files
belongs to whatever that project ships, and handing it over wires this
project to the wrong account.

## `ready: false` → wire the tag, then connect

Your one edit. Put the tag the script printed into whatever produces the
page head, dev-gated by the project's own idiom (`NODE_ENV ===
"development"`, a dev template block, a server dev flag); no dev/prod split
→ plain tag plus one line to the user that it ships until removed.

Ask the user which file if it isn't obvious from what you already have
open. Don't go looking — a wrong file here is worse than a question.

Then set `ready: true` in `.claude/pixy.json` and add `"tag": "<the
file>"` (uninstall needs it). Tell the user to restart their dev server if
hot reload skips template or config files. Then connect.

## `ready: true` → connect

1. Open `url` in the browser and show the user the link — they're switching
   there now.
2. Call `pixy_wait` with `all: true` once; leftovers from a previous
   session get one compact mention via `pixy_say`. Then loop. If pairing
   happened in this same session, pass `key: "<the key>"` on every pixy
   tool call — the header takes over after the user's next restart.

## The loop — design first, build on "go"

The user designs; you build **when asked**. Keep the terminal quiet.

- **Visual edit records** (style/move/html/add/delete/comment/…) are the
  user designing. They arrive bundled with chat as context. Do NOT implement
  on arrival, do not ack, do not comment on each one.
- **Chat records** (`op: "chat"`, text in `to`; `target` = about that
  element at `viewport.width`): answer with `pixy_say`, ack the chat record.
  Talking about a design is not permission to build it.
- **The "go"** ("do it", "apply this"): implement the accumulated records
  (or the named subset), mapping `viewport.width` to the project's
  breakpoints. `pixy_say` `kind: "progress"` chunks as you go. Ack
  implemented `done`, declined `skipped` with a reason.
- **Mirror each exchange in the terminal** as two compact quote lines
  (`> user: …` / `> me: …`) — the session must be readable from the Claude
  side afterwards. No other terminal chatter while live.
- **Inspect with Read/Grep/Glob, not Bash pipelines** — dedicated tools
  don't prompt for permission.
- **Interrupts** (PreToolUse denials): `INTERRUPT from Pixy: …` → stop the
  current approach, call `pixy_wait` for direction. `Message from Pixy user
  (continue working, incorporate): …` → do NOT abort, fold it in.
- **Ending**: user asks to end (either side) → `pixy_end`, exit, summarize
  in the terminal. `session: "ended"` in a result → same without
  `pixy_end`. Empty `pixy_wait` just re-arms — call it again; never stop or
  ask "anything else?" while live.
- `delivered: false` from `pixy_say` → no page connected; say it in the
  terminal.
- The page hot-reloading on your saves is normal; the session survives it,
  and survives relay restarts (the panel reconnects itself).
