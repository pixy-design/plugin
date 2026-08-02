---
description: Remove Pixy from this project — script tag, settings entries, state file
---

This file is the COMPLETE uninstall instruction — do not load any skill or
read plugin files.

1. Remove the script tag from the file recorded in `.claude/pixy.json`
   (`tag`); if `"manual"`, tell the user where they put it.
2. From `~/.claude/settings.json` remove: the two `mcp__…` allow rules,
   `enabledPlugins["pixy@pixy"]`, `extraKnownMarketplaces.pixy`, and
   `env.PIXY_ORIGIN` if present. Leave everything else untouched. Pixy
   writes nothing into the project's own `settings.json`, so there is
   nothing to clean there — if you find `PIXY_KEY` in one, it predates
   this version; remove it and say so.
3. Delete `.claude/pixy.json` (and `.claude/pixy-connect` or
   `.claude/pixy-pair.json` if an older or interrupted run left them).
4. Say: the plugin unloads on the next restart; the project key still
   exists at https://pixydesignapp.com/@my — delete it there to kill old
   tags.
