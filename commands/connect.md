---
description: Start a Pixy live session — design in your page, Claude builds on "go"
---

This file is the COMPLETE connect instruction. Do not load any skill, do not
read plugin or skill files, never search `~/.claude` or the plugin cache.
Connect is a cold start: **three tool calls, pixy_wait armed in ~15 seconds.**

1. Run `bash .claude/pixy-connect` (allowlisted, written by setup). It
   prints the project state (url, server, tag, origin) and `SERVER_UP` or
   `SERVER_DOWN`.
   - `NO_STATE` → this project isn't set up: invoke the `pixy:setup` skill,
     then come back here.
   - `SERVER_DOWN` and `server` is a command → start it (background), go on.
   - `SERVER_DOWN` and `server` is `"external"` → one line asking the user
     to start their server; re-run the preflight when they say it's up.
2. Open `url` in the browser — the user is switching there now.
3. Call `pixy_wait` with `all: true` once; leftovers from a previous session
   get one compact mention via `pixy_say`. Then loop. If pairing happened in
   this same session, pass `key: "<the key>"` on every pixy tool call.

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
