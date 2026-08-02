---
description: Send a message to the Pixy team — questions, feedback, requests
---

This file is the COMPLETE instruction — do not load any skill or read plugin
files. **Read `.claude/pixy.json` only**, for `origin`, `key` and `url`.

For something that is broken, `/pixy:bug` is the better door — it attaches
what you saw in the terminal. This one is for everything else: a question,
a feature request, a thought about how it works.

## 1. Get the message

`$ARGUMENTS` is the message. Send it as written — you are the envelope, not
the author; do not rewrite, expand or make it more polite. Empty → ask what
they'd like to say, in one line, and wait.

Add nothing of your own. Unlike a bug report there is no terminal context
worth attaching here, and a message the user didn't write is one they can't
stand behind.

## 2. Send it

Write the payload to `.claude/pixy-report.json` (the Write tool, so the
message survives quotes and newlines intact):

```json
{
	"method": "contact.send",
	"kind": "contact",
	"key": "<key from pixy.json>",
	"url": "<url from pixy.json, or omit>",
	"message": "<the message>"
}
```

```bash
curl -s -m 15 -X POST "$(python3 -c 'import json;print(json.load(open(".claude/pixy.json"))["origin"])')/" -H "Content-Type: application/json" -d @.claude/pixy-report.json
```

Then delete `.claude/pixy-report.json` — it holds the key.

No `.claude/pixy.json` → no key to send with. Point them at
https://pixydesignapp.com/@my and stop.

## 3. Report back

`"ok": true` → one line: sent, and the project it came from. Anything else →
say it failed and offer the message back so they can send it themselves.
Never retry in a loop.
