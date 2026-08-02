---
description: What Pixy is, and whether this project is wired
---

This file is the COMPLETE instruction. Print a short card and stop — do not
load any skill, do not read plugin files, do not search the project.

**Read `.claude/pixy.json` and nothing else.** Change nothing, start
nothing, and do not offer to connect — the user asked what this is, not to
run it.

Print, in this order:

1. **What it is** — a line or two: design live in your running page; visual
   edits and chat come back to Claude as records; Claude implements them in
   source when you say "go".
2. **This project** — from `.claude/pixy.json`:
   - no file → "not wired yet — `/pixy:connect` sets it up"
   - `ready: false` → "paired, but the script tag isn't placed yet"
   - `ready: true` → "wired", plus the `url` as a link and the `tag` file
   **Never print the key**, not even partially.
3. **Commands** — `/pixy:connect` to start a session, `/pixy:disable` to
   unwire this project, `/pixy:uninstall` to remove Pixy entirely.

Then stop. No "want me to connect?" — if they do, they'll say so.
