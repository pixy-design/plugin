---
description: Uninstall Pixy everywhere — this project's tag plus the plugin itself
---

This file is the COMPLETE uninstall instruction — do not load any skill or
read plugin files. **This removes the plugin from Claude Code, not just
from this project.** If the user only wants this project unwired with the
plugin left installed, that is `/pixy:disable` — say so and stop.

1. Remove the script tag from the file recorded in `.claude/pixy.json`
   (`tag`); if `"manual"`, tell the user where they put it.
2. From `~/.claude/settings.json` remove the four allow rules Pixy added —
   two `mcp__…`, two `Bash(claude plugin …)` — plus
   `enabledPlugins["pixy@pixy"]`, `extraKnownMarketplaces.pixy`, and
   `env.PIXY_ORIGIN` if present. Leave everything else untouched. Pixy
   writes nothing into the project's own `settings.json`, so there is
   nothing to clean there — if you find `PIXY_KEY` in one, it predates
   this version; remove it and say so.
3. Delete `.claude/pixy.json` (and `.claude/pixy-connect`,
   `.claude/pixy-pair.json` or `.claude/pixy-report.json` if an older or
   interrupted run left them — the last two hold a key).
4. Say: the plugin unloads on the next restart; other projects still
   carrying a tag will go inert with it gone. The project key still exists
   at https://pixydesignapp.com/my — delete it there to kill old tags.
