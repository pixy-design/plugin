---
name: pixy
description: Live design session with the user inside their web page. Use when the user says "continue in Pixy", "let's pixy", runs /pixy (setup | connect | uninstall), or wants to discuss/edit their running site visually. Drives the pixy_wait loop against the Pixy relay.
---

# Pixy live session

The user is in the browser, on their own running site with the Pixy script
active. The conversation continues **there**, not in the terminal: their chat
messages arrive as records, your replies appear in their page's panel, and
their visual edits accumulate as design context. The terminal is meant to go
quiet.

Three flows: **setup** (once per project), **connect** (every session),
**uninstall**. `/pixy` with no argument means connect; connect without a
completed setup runs setup first, then connects in the same turn.

## The state file

Setup writes `.claude/pixy.json`; connect reads it and asks nothing:

```json
{
	"url": "http://localhost:3000",
	"server": "npm run dev",
	"tag": "src/app/layout.tsx",
	"origin": "https://pixydesignapp.com"
}
```

`url` — the page carrying the script; `server` — how the dev server starts
(`"external"` if the user runs it themselves); `tag` — the file holding the
script tag (`"manual"` if the user placed it); `origin` — the Pixy host the
tag loads from.

## Setup — you do the wiring, never the user

Do not walk the user through editing configs or visiting dashboards — write
the files, run the commands, verify. Ask nothing that the repo can answer.

1. **Key.** Use, in order: `PIXY_KEY` in the session env; `env.PIXY_KEY` in
   `.claude/settings.json`; a key the user pasted ("key px_…"). Otherwise
   pair: call `pixy_pair` with context (folder name, git remote, dev URL),
   show the returned link — one line: "Connect this project to your Pixy
   account: {link}" — and poll `pixy_pair` with the returned `code` +
   `secret` every ~5s until `status: "approved"` hands you the key. The link
   expires in 10 minutes; on expiry start over once, then fall back to
   asking for the key (https://pixydesignapp.com/@my).
2. **Settings.** Merge into `.claude/settings.json` (create if absent,
   preserve everything already there):

   ```json
   {
   	"env": { "PIXY_KEY": "<the key>" },
   	"permissions": {
   		"allow": [
   			"mcp__plugin_pixy_pixy__*",
   			"Bash(<dev server command>:*)",
   			"mcp__Claude_Browser__*"
   		],
   		"defaultMode": "acceptEdits"
   	}
   }
   ```

   `<dev server command>` is this project's real one (`npm run dev`,
   `./run`, …) — the allowlist is why a live session doesn't rain
   permission prompts.
3. **Script tag.** Install it in the project's source yourself — this is the
   step that makes the page a Pixy canvas:

   ```html
   <script type="module" src="{origin}/pixy.mjs?k={key}"></script>
   ```

   `{origin}` is `https://pixydesignapp.com` for `px_` keys,
   `https://sandbox.pixydesignapp.com` for `pxs_` keys. First grep the
   source for an existing `pixy.mjs` tag — if one is there, update its key
   if stale and record its location, never add a second. Otherwise put it
   in the head
   of whatever produces the page's HTML (index.html, the root layout, the
   base template) and **gate it to development builds** using the project's
   own idiom (`process.env.NODE_ENV === "development"`, a dev-only template
   block, a server-side dev flag). If the project has no dev/prod split
   (plain static site), add the plain tag and tell the user in one line that
   it ships until removed.
4. **Dev server.** Start it — or restart it if the tag edit isn't picked up
   by hot reload (template/config files often aren't). Record the command
   and URL for the state file. If the server is the user's own long-running
   process you shouldn't touch, set `server: "external"` and say one line:
   "restart your dev server to pick up the tag".
5. **Verify, don't assume.** Fetch the dev URL and confirm the tag is in the
   served HTML; fetch `{origin}/pixy.mjs?k={key}` and confirm it serves the
   real module (the keyless stub says "needs a personal key" — that means a
   bad or stale key). Fix what fails before declaring done.
6. **State file.** Write `.claude/pixy.json`.
7. **Hand off.** One closing message: setup is done, and one restart of
   Claude Code arms the key and the permission rules (env and hooks
   snapshot at session start) — then `/pixy` starts a session. Setup ends
   here; do not open the loop in the same run unless the user already
   restarted once before (key was in env at session start), in which case
   offer to connect right away.

## Connect — the live session

1. Read `.claude/pixy.json`. Missing → run Setup above first. Key present in
   env or settings; if the pairing happened this session, pass
   `key: "<the key>"` on every pixy tool call (the header takes over after
   the next restart).
2. Ensure the dev server is running (the `server` command; `external` means
   check the URL responds and, if not, ask the user to start it — one line).
3. Open `url` in the browser so the user can switch straight to it.
4. Call `pixy_wait` with `all: true` once — this returns anything
   outstanding from before (records delivered earlier but never resolved).
   If there are leftovers, say what they are in the panel and whether
   anything needs finishing.
5. Enter the loop: `pixy_wait`, handle, `pixy_wait` again immediately.

## The loop — design first, build on "go"

The user designs; you build **when asked**. Their workflow is a design pass
— moving, restyling, annotating — followed by "do it". Respect the phases:

- **Visual edit records** (style/move/html/add/delete/comment/…) are the
  user designing. They arrive bundled with chat messages as context. Do NOT
  implement them on arrival, do not ack them, do not comment on each one —
  they accumulate, on the relay and in your context.
- **Chat records** (`op: "chat"`, text in `to`; a `target` means it's about
  that element at `viewport.width`) are the user talking to you. Answer with
  `pixy_say`, then `pixy_ack` the chat record. Questions get answers;
  discussion gets discussion — talking about a design is not permission to
  build it.
- **The "go"** — the user asks for implementation ("do it", "apply this",
  "build the header changes"): implement the accumulated records (or the
  named subset), mapping each record's `viewport.width` to the project's
  breakpoints. Send `kind: "progress"` chunks via `pixy_say` as you go — not
  one blob at the end. Ack every record you implemented `done`, declined
  ones `skipped` with a reason.
- An empty `pixy_wait` just re-arms; call it again. Do not stop, do not
  summarize, do not ask "anything else?" in the terminal while the session
  is live.
- When a result says `session: "ended"` — the user pressed Done, closed the
  page, or went idle — exit the loop and summarize everything that changed
  in the terminal.

## Interrupts

While you work, the PreToolUse gate may deny a tool call with a message:

- `INTERRUPT from Pixy: …` — stop the current approach, call `pixy_wait`
  for direction.
- `Message from Pixy user (continue working, incorporate): …` — do NOT
  abort; fold the guidance into what you're doing.

## Uninstall

1. Remove the script tag from the file recorded in `.claude/pixy.json`
   (`tag`); if `manual`, tell the user where they put it.
2. From `.claude/settings.json` remove: `env.PIXY_KEY`, the three allow
   rules setup added, `enabledPlugins["pixy@pixy"]`, and
   `extraKnownMarketplaces.pixy`. Leave everything else — including
   `defaultMode` — untouched.
3. Delete `.claude/pixy.json`.
4. Say: the plugin unloads on the next restart; the project key still exists
   at https://pixydesignapp.com/@my — delete it there to kill old tags.

## Notes

- `delivered: false` from `pixy_say` means no page is connected — say it in
  the terminal instead.
- The page hot-reloads when you save files — the user sees your edits land
  live. That reload is normal; the session survives it, and it survives a
  relay restart too (the panel reconnects and resumes by itself).
