---
description: Remove Pixy from this project — script tag, settings entries, state file
---

This file is the COMPLETE uninstall instruction — do not load any skill or
read plugin files.

1. Remove the script tag from the file recorded in `.claude/pixy.json`
   (`tag`); if `"manual"`, tell the user where they put it.
2. From `.claude/settings.json` remove: `env.PIXY_KEY`, the allow rules
   setup added, `enabledPlugins["pixy@pixy"]`, and
   `extraKnownMarketplaces.pixy`. Leave everything else — including
   `defaultMode` — untouched.
3. Delete `.claude/pixy.json` and `.claude/pixy-connect`.
4. Say: the plugin unloads on the next restart; the project key still
   exists at https://pixydesignapp.com/@my — delete it there to kill old
   tags.
