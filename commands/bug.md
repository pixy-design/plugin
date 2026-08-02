---
description: Report a Pixy bug — goes straight to the people who build it
---

This file is the COMPLETE instruction — do not load any skill or read plugin
files. **Read `.claude/pixy.json` only**, for `origin`, `key` and `url`.

## 1. Get the report

The user's own words are the report. If `$ARGUMENTS` has them, use them as
written — do not rewrite, summarize or "improve" the wording. If it's empty,
ask what went wrong in one line and wait.

Then append what you saw yourself, under a `---` line: the command or action
that broke, the error text verbatim, what you expected instead. You were
there and the user wasn't watching the terminal; that half is usually the
half that gets it fixed. Include no file contents, no code, no key.

## 2. Send it

Write the payload to `.claude/pixy-report.json` (the Write tool, so the
message survives quotes and newlines intact):

```json
{
	"method": "contact.send",
	"kind": "bug",
	"key": "<key from pixy.json>",
	"url": "<url from pixy.json, or omit>",
	"message": "<the report>"
}
```

```bash
curl -s -m 15 -X POST "$(python3 -c 'import json;print(json.load(open(".claude/pixy.json"))["origin"])')/" -H "Content-Type: application/json" -d @.claude/pixy-report.json
```

Then delete `.claude/pixy-report.json` — it holds the key.

No `.claude/pixy.json` → no key to send with. Say the report needs a wired
project (`/pixy:connect`), and stop; do not send it unattributed.

## 3. Report back

`"ok": true` → one line: sent, and the project it came from. Anything else →
say it failed and offer the message back so the user can paste it at
https://pixydesignapp.com/@my. Never retry in a loop.
