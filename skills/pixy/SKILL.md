---
name: pixy
description: Live design session with the user inside their web page. Use when the user says "continue in Pixy", "let's pixy", or wants to discuss/edit their running site visually.
---

# Pixy

Everything lives in this plugin's self-contained commands — invoke the
matching skill with the Skill tool and follow it exactly. Do not search for
skill or plugin files.

- `pixy:connect` — start the live session (the default: "let's pixy",
  "continue in Pixy" mean this). Wires the project first by itself if it
  has no `.claude/pixy.json`.
- `pixy:info` — what Pixy is and whether this project is wired. Read-only.
- `pixy:update` — update the plugin, reconcile this project's wiring.
- `pixy:bug` — report something broken; attaches what went wrong here.
- `pixy:contact` — send the Pixy team a question or a request.
- `pixy:disable` — unwire this project (tag, state file); plugin stays.
- `pixy:uninstall` — remove Pixy from the project *and* from Claude Code.
