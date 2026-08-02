---
description: Unwire Pixy from this project — tag and state file; plugin stays installed
---

This file is the COMPLETE instruction — do not load any skill or read
plugin files. **This project only.** The plugin stays installed and other
projects keep working; `~/.claude/settings.json` is not touched. If the
user wants Pixy gone from Claude Code entirely, that is `/pixy:uninstall` —
say so and stop rather than doing half of each.

1. Remove the script tag from the file recorded in `.claude/pixy.json`
   (`tag`); if `"manual"`, tell the user where they put it.
2. Delete `.claude/pixy.json` (and `.claude/pixy-connect`,
   `.claude/pixy-pair.json` or `.claude/pixy-report.json` if an older or
   interrupted run left them — the last two hold a key).
3. Say: `/pixy:connect` wires it again from scratch; the project key still
   exists at https://pixydesignapp.com/@my — delete it there if you want
   any tag still carrying it to go dead.
