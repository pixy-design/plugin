---
description: Start a Pixy live session — design in your page, Claude builds on "go"
---

This file is the COMPLETE instruction for connecting AND for wiring a
project that isn't wired yet. Do not load any skill, do not read plugin or
skill files, never search `~/.claude` or the plugin cache.

**Read `.claude/pixy.json` and branch on it. That file is the only thing
you inspect before acting.** No searching the project, no reading
`package.json`, no grepping for `pixy.mjs`, no looking for a dev command or
a URL — every one of those answers is in that file or comes from the user.
There is nothing to discover.

`"v"` is that file's shape. Missing or `1` is the shape below — an older
file that predates the field is a `1`, so read straight on. Higher than `1`
means a newer Pixy wrote it: do not guess at the fields and do not repair
it, just say the plugin is behind and send them to `/pixy:update`.

## No file → pair

**Ask nothing.** Not the URL, not the dev command, not which file holds the
head. Run, from the project root:

```bash
curl -fsSL https://mcp.pixydesignapp.com/setup | bash -s --
```

**`PAIRING <link>`** → show the user the one line it printed, then re-run
the same command plus `--wait` (give the Bash call a 600000ms timeout).
That run blocks on their approval and writes `.claude/pixy.json`.
`PAIR_PENDING` → one line nudging them, re-run with `--wait`.
`PAIR_UNAVAILABLE` → ask for a key from the dashboard, re-run with `--key
<the key>`.

**Only ever pass `--key` with a key the user typed into chat.** Never one
you found while looking around — a `px_…` in the project's own files
belongs to whatever that project ships, and handing it over wires this
project to the wrong account. The key you were given lives in exactly two
places: `.claude/pixy.json`, and the tag. Never copy it into
`settings.json`, an env var, a `.env`, or any config of the project's own.

Then continue at **`ready: false`**.

## `ready: false` → wire the tag, then connect

Your one edit. Put the tag the script printed into whatever produces the
page head, dev-gated by the project's own idiom (`NODE_ENV ===
"development"`, a dev template block, a server dev flag); no dev/prod split
→ plain tag plus one line to the user that it ships until removed.

You're the one working in this codebase — you know which file renders the
head and which URL the project serves on. Decide, don't ask, and don't turn
it into an expedition: the answer is in what you already have open, or one
look at the obvious file.

Then, in the same breath, write into `.claude/pixy.json`:

- `"url"` — the page you'll tell the user to open
- `"tag"` — the file you just edited, so uninstall is a clean removal and
  not a hunt
- `"ready": true`

Tell the user to restart their dev server if hot reload skips template or
config files. Then connect.

## `ready: true` → connect

1. Give the user the `url` as a link to follow, and offer to open it in the
   browser. **If they decline the browser, that is not a signal about
   anything** — they are opening it themselves, which was always the more
   likely thing. Say nothing about it and carry straight on to step 2.
   **Do not check whether the tag is really on the page** — no curl, no
   fetch, no grep. It was wired; if something is off the user sees it on
   the page in front of them, which is faster than any check you could run.
2. Call `pixy_wait` with `all: true` once; leftovers from a previous
   session get one compact mention via `pixy_say`. Then loop. Pass `key:
   "<the key from pixy.json>"` on every pixy tool call — that file is the
   only place it lives, so it never becomes ambient.

**`session: "none"` or `"ended"` before the user has opened the page means
exactly that: they haven't opened it yet.** It is not the session ending —
there was no session. Keep `pixy_wait` armed and wait for them; the call
parks and returns the moment their panel connects. Never announce that
there's nothing to do and stop. You are early, not finished.

## The loop — design first, build on "go"

The user designs; you build **when asked**. Keep the terminal quiet.

- **Visual edit records** (style/move/html/add/delete/comment/…) are the
  user designing. They arrive bundled with chat as context. Do NOT implement
  on arrival, do not ack, do not comment on each one.
- **Chat records** (`op: "chat"`, text in `to`; `target` = about that
  element at `viewport.width`): answer with `pixy_say`, ack the chat record.
  Talking about a design is not permission to build it.
- **The "go"** ("do it", "apply this"): implement the accumulated records
  (or the named subset), mapping `viewport.width` to the project's
  breakpoints. `pixy_say` `kind: "progress"` chunks as you go. Ack
  implemented `done`, declined `skipped` with a reason.
- **Mirror each exchange in the terminal** as two compact quote lines
  (`> user: …` / `> me: …`) — the session must be readable from the Claude
  side afterwards. No other terminal chatter while live.
- **Inspect with Read/Grep/Glob, not Bash pipelines** — dedicated tools
  don't prompt for permission.
- **Interrupts** (PreToolUse denials): `INTERRUPT from Pixy: …` → stop the
  current approach, call `pixy_wait` for direction. `Message from Pixy user
  (continue working, incorporate): …` → do NOT abort, fold it in.
- **Ending**: user asks to end (either side) → `pixy_end`, exit, summarize
  in the terminal. `session: "ended"` **after the session has been live in
  this run** → same without `pixy_end`; before that it only means the page
  isn't open yet (see above). Empty `pixy_wait` just re-arms — call it
  again; never stop or ask "anything else?" while live.
- `delivered: false` from `pixy_say` → no page connected; say it in the
  terminal.
- The page hot-reloading on your saves is normal; the session survives it,
  and survives relay restarts (the panel reconnects itself).
