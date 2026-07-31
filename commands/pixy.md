---
description: Pixy live design — /pixy setup · /pixy connect (default) · /pixy uninstall
argument-hint: "[setup | connect | uninstall]"
---

Load the `pixy` skill (from this plugin) and run the flow named by the
argument: **$ARGUMENTS**

- `setup` — wire this project end to end: pairing, settings + permissions,
  the script tag in the project's source, dev server restart, verification.
  One-time, ends with a single Claude Code restart.
- `connect` (or no argument) — start the live session: read the project's
  `.claude/pixy.json`, open the page, enter the `pixy_wait` loop and stay in
  it. No discovery, no questions — setup already answered them.
- `uninstall` — remove the script tag, the Pixy settings entries, and the
  state file from this project.

No argument means `connect`. If connect finds no `.claude/pixy.json`, run the
setup flow first, then connect in the same turn. The user is switching to the
browser; keep the terminal quiet while the session is live.
