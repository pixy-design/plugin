---
description: Continue this session live inside your page — chat and design together on the running site
---

Start a Pixy live session: load the `pixy` skill (from this plugin) and follow
its protocol — ensure the dev server runs, open the page carrying pixy.mjs in
the browser, then enter the `pixy_wait` loop and stay in it until the session
ends. The user is switching to the browser now; keep the terminal quiet while
the session is live.

If `PIXY_KEY` is not set in this session, follow the skill's Setup section
first: you write the project's `.claude/settings.json` (key + permissions)
yourself and tell the user to restart — never send them off to edit configs.
