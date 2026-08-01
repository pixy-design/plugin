---
name: pixy
description: Live design session with the user inside their web page. Use when the user says "continue in Pixy", "let's pixy", or wants to discuss/edit their running site visually.
---

# Pixy

Everything lives in this plugin's three self-contained commands — invoke the
matching skill with the Skill tool and follow it exactly. Do not search for
skill or plugin files.

- `pixy:connect` — start the live session (the default: "let's pixy",
  "continue in Pixy" mean this). Runs setup first by itself if the project
  has no `.claude/pixy.json`.
- `pixy:setup` — one-time project wiring (pairing, settings, script tag).
- `pixy:uninstall` — remove Pixy from the project.
