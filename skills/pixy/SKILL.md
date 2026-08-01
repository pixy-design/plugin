---
name: pixy
description: Live design session with the user inside their web page. Use when the user says "continue in Pixy", "let's pixy", runs /pixy (setup | connect | uninstall), or wants to discuss/edit their running site visually. Drives the pixy_wait loop against the Pixy relay.
---

# Pixy live session

The user is in the browser, on their own running site with the Pixy script
active. The conversation continues **there**: their chat messages arrive as
records, your replies appear in their page's panel, and their visual edits
accumulate as design context.

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

## Setup — fast, you do the wiring

Setup is two and a half moves, not an investigation. Target: **under two
minutes of your own work**; the pairing approval is the user's time, and you
work through it, never idle. Do not explore the repo, do not read files
beyond this list, do not ask questions the repo answers.

1. **One discovery pass, all in parallel** (single batch of tool calls):
   read `package.json` (or launch config / Procfile / run script) for the
   dev command and port; Grep the source for an existing `pixy.mjs` tag;
   Glob for where the page head is produced (`index.html`, root layout,
   base template). That is ALL the discovery setup needs.
2. **Key**: session env `PIXY_KEY` → `env.PIXY_KEY` in
   `.claude/settings.json` → a key in the user's message. None → call
   `pixy_pair` (context: folder, git remote, dev URL), show the link — one
   line: "Connect this project to your Pixy account: {link}" — and **keep
   working while it's pending**: do steps 3–4 now, polling `pixy_pair`
   (code + secret) between actions, ~5s apart. Link dead after 10 min →
   restart pairing once, then ask for the key
   (https://pixydesignapp.com/@my).
3. **Settings**: merge into `.claude/settings.json` (create if absent,
   preserve what's there) — the allowlist is why sessions run prompt-free:

   ```json
   {
   	"env": { "PIXY_KEY": "<the key>" },
   	"permissions": {
   		"allow": [
   			"mcp__plugin_pixy_pixy__*",
   			"mcp__Claude_Browser__*",
   			"Bash(<dev server command>:*)",
   			"Bash(ls:*)", "Bash(cat:*)", "Bash(head:*)", "Bash(tail:*)",
   			"Bash(grep:*)", "Bash(rg:*)", "Bash(find:*)", "Bash(wc:*)",
   			"Bash(curl -s:*)",
   			"Bash(git status:*)", "Bash(git diff:*)", "Bash(git log:*)"
   		],
   		"defaultMode": "acceptEdits"
   	}
   }
   ```

   Append the project's own routine commands (its build, its check —
   `node --check`, `npm test`, `python3 -m …`) — whatever a session on this
   codebase will obviously run.
4. **Script tag**: if step 1 found an existing `pixy.mjs` tag, update its
   key if stale and record its location — never add a second. Otherwise
   insert into the head, dev-gated by the project's own idiom
   (`NODE_ENV === "development"`, a dev template block, a server dev flag);
   no dev/prod split (plain static site) → plain tag + one line to the user
   that it ships until removed:

   ```html
   <script type="module" src="{origin}/pixy.mjs?k={key}"></script>
   ```

   `{origin}`: `https://pixydesignapp.com` for `px_` keys,
   `https://sandbox.pixydesignapp.com` for `pxs_` keys.
5. **Verify with two fetches, fix what fails**: the dev URL serves HTML
   containing the tag (start/restart the dev server if needed — hot reload
   often skips template/config files); `{origin}/pixy.mjs?k={key}` serves
   the real module (a stub mentioning "personal key" = bad/stale key). A
   user-managed server you shouldn't touch → `server: "external"`, one
   line: "restart your dev server to pick up the tag".
6. **Finish**: write `.claude/pixy.json`; one closing message — setup done,
   restart Claude Code once to arm the key and permissions, then `/pixy`.
   Do not open the loop now unless the key was already armed at session
   start (then offer to connect right away).

## Connect — cold start, not an investigation

Target: **first `pixy_wait` within ~15 seconds.**

1. Read `.claude/pixy.json` — it is the entire truth. Do not re-derive it,
   do not read other project files, do not verify the tag again. Missing →
   run Setup first.
2. One `curl -sI` against `url`: server up → go; down and `server` is a
   command → start it; down and `"external"` → one line asking the user to
   start it.
3. Open `url` in the browser, call `pixy_wait` with `all: true` once —
   leftovers from a previous session get one compact mention in the panel.
4. Enter the loop.

If pairing happened this very session, pass `key: "<the key>"` on every
pixy tool call — the env header takes over after the next restart.

## The loop — design first, build on "go"

The user designs; you build **when asked**. Their workflow is a design pass
— moving, restyling, annotating — followed by "do it".

- **Visual edit records** (style/move/html/add/delete/comment/…) are the
  user designing. They arrive bundled with chat messages as context. Do NOT
  implement them on arrival, do not ack them, do not comment on each one.
- **Chat records** (`op: "chat"`, text in `to`; a `target` means it's about
  that element at `viewport.width`) are the user talking to you. Answer
  with `pixy_say`, then `pixy_ack` the chat record. Talking about a design
  is not permission to build it.
- **The "go"** — the user asks for implementation ("do it", "apply this"):
  implement the accumulated records (or the named subset), mapping each
  record's `viewport.width` to the project's breakpoints. Send
  `kind: "progress"` chunks via `pixy_say` as you go. Ack implemented
  records `done`, declined ones `skipped` with a reason.
- **Mirror the conversation in the terminal.** After each exchange, emit
  the pair as two compact quote lines (`> user: …` / `> me: …`) — nothing
  more. The session must be readable from the Claude side afterwards, so
  the user can close the page and continue here with full history. No other
  terminal chatter while live.
- **Use dedicated tools, not Bash, for inspection** — Read/Grep/Glob don't
  prompt for permission; ad-hoc `ls | grep` pipelines do.
- **Ending**: the user asking you to end/close the session (either side) →
  call `pixy_end`, exit the loop, summarize. A result with
  `session: "ended"` (Done pressed, page closed, idle) → same, without
  `pixy_end`. An empty `pixy_wait` just re-arms — call it again; never stop
  or ask "anything else?" in the terminal while live.

## Interrupts

While you work, the PreToolUse gate may deny a tool call with a message:

- `INTERRUPT from Pixy: …` — stop the current approach, call `pixy_wait`
  for direction.
- `Message from Pixy user (continue working, incorporate): …` — do NOT
  abort; fold the guidance into what you're doing.

## Uninstall

1. Remove the script tag from the file recorded in `.claude/pixy.json`
   (`tag`); if `manual`, tell the user where they put it.
2. From `.claude/settings.json` remove: `env.PIXY_KEY`, the allow rules
   setup added, `enabledPlugins["pixy@pixy"]`, and
   `extraKnownMarketplaces.pixy`. Leave everything else — including
   `defaultMode` — untouched.
3. Delete `.claude/pixy.json`.
4. Say: the plugin unloads on the next restart; the project key still
   exists at https://pixydesignapp.com/@my — delete it there to kill old
   tags.

## Notes

- `delivered: false` from `pixy_say` means no page is connected — say it in
  the terminal instead.
- The page hot-reloads when you save files — the user sees your edits land
  live. That reload is normal; the session survives it, and it survives a
  relay restart too (the panel reconnects and resumes by itself).
