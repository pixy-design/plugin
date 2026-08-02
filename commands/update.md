---
description: Update the Pixy plugin and reconcile this project's wiring
---

This file is the COMPLETE instruction — do not load any skill or read plugin
files. Two commands and a short report; nothing else. This is not a repair
command: never reinstall, never re-pair, never touch `settings.json`.

## 1. Update the plugin

```bash
claude plugin marketplace update pixy
```

```bash
claude plugin update pixy@pixy
```

The marketplace refresh comes first — without it the update checks stale
metadata and reports "already current" against yesterday's version. Already
current after a real refresh → say so in one line and go to step 2 anyway.

## 2. Reconcile this project

**Read `.claude/pixy.json` and nothing else.** Branch on it:

- **no file** → the plugin updated, this project was never wired.
  `/pixy:connect`. Stop.
- **`ready: false`** → paired, tag never placed. `/pixy:connect`. Stop.
- **`v` missing, or `v: 1`** → current shape, nothing to migrate. The tag is
  a bare URL, so the module it serves is already whatever the server has —
  there is no tag to re-place and no version in it to bump.
- **`v` greater than 1** → this file was written by a newer Pixy than the
  one now installed. Do not guess at the fields and do not rewrite it; say
  the update didn't take and the restart in step 3 is what applies it.

## 3. Restart

Say it plainly: the new plugin loads on the next Claude Code start, not in
this session. Nothing else to do.
