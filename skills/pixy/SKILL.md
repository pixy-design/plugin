---
name: pixy
description: Live design session with the user inside their web page. Use when the user says "continue in Pixy", "let's pixy", runs /pixy, or wants to discuss/edit their running site visually. Drives the pixy_wait loop against the Pixy relay.
---

# Pixy live session

The user is in the browser, on their own running site with the Pixy script
active. The conversation continues **there**, not in the terminal: their chat
messages and visual edits arrive as records; your replies appear in their
page's panel. The terminal is meant to go quiet.

## Setup — you do the wiring, never the user

If `PIXY_KEY` is missing from the session env (or the pixy tools report "no
valid Pixy key"): the plugin is installed but the project isn't connected yet.

1. **If the user gave a key in their message** ("let's work with Pixy, key
   px_…") — skip to step 3 with it.
2. **Otherwise, pair**: call `pixy_pair` with context (project folder name,
   git remote, dev URL if known). Show the returned link to the user — one
   line, e.g. "Connect this project to your Pixy account: {link}" — and poll
   `pixy_pair` with the returned `code` + `secret` every ~5 seconds until
   `status: "approved"` hands you the key. The link expires in 10 minutes;
   on expiry, start over once, then fall back to asking for the key
   (https://pixydesignapp.com/@my).
3. Merge into the project's `.claude/settings.json` yourself (create if
   absent, preserve existing keys):

   ```json
   {
   	"env": { "PIXY_KEY": "<the key>" },
   	"permissions": {
   		"allow": ["mcp__plugin_pixy_pixy__*"],
   		"defaultMode": "acceptEdits"
   	}
   }
   ```

4. **Keep working immediately** — do not end the turn for a restart. Pass
   `key: "<the key>"` as an argument on every pixy tool call for the rest of
   this session; the `PIXY_KEY` header takes over after the user's next
   restart. Mention once: "Wired up — a restart later will also arm the Stop
   button" (the interrupt hook reads env, which snapshots at session start).

Do not walk the user through editing config files — write them. Do not ask
which settings they want — these are the settings.

## Protocol

1. Ensure the project's dev server is running, then open the project URL —
   the page that carries `pixy.mjs`. Infer the URL from the repo (launch
   config, package scripts, listening port). Ask only if genuinely ambiguous.
   Do not present a picker.
2. Call `pixy_wait`.
3. For each returned item:
   - `op: "chat"` — the user's message (`to` is the text; a `target` means it
     is about that element, at `viewport.width`). Answer or act. Reply with
     `pixy_say` — during long work send `kind: "progress"` chunks as you go,
     not one blob at the end. `pixy_ack` when handled.
   - any other op — a visual edit record: implement it in the source
     (map `viewport.width` to the project's breakpoints), then `pixy_ack`
     `done`, or `skipped` with a reason.
4. Call `pixy_wait` again **immediately**. Staying in this loop is correct —
   an empty result just re-arms. Do not stop, do not summarize, do not ask
   "anything else?" in the terminal while the session is live.
5. When a result says `session: "ended"` — the user pressed Done, closed the
   page, or went idle — exit the loop and summarize everything that changed
   in the terminal.

## Interrupts

While you work, a PreToolUse gate may deny a tool call with a message from
the user:

- `INTERRUPT from Pixy: …` — stop the current approach, call `pixy_wait`
  for direction.
- `Message from Pixy user (continue working, incorporate): …` — do NOT
  abort; fold the guidance into what you're doing.

## Notes

- `delivered: false` from `pixy_say` means no page is connected — say it in
  the terminal instead.
- Records you don't ack are re-delivered by every `pixy_wait`; ack everything
  you handled.
- The page hot-reloads when you save files — the user sees your edits land
  live. That reload is normal, the session survives it.
