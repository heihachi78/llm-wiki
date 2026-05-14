# Active-KB Workflow

Use when the user asks to inspect or switch the active KB.

This workflow does not modify wiki content, QMD indexes, or any KB folder. It only reads or writes `.agents/active-kb`.

## Inspect

When no KB name is provided:

1. Detect KBs by globbing `*/wiki-config.json` and `*/*/wiki-config.json` from the project root.
2. Read `.agents/active-kb` if present. If missing, read `.claude/active-kb` as a legacy fallback and label it as such.
3. Print:
   - Active KB and absolute path if valid.
   - Stale pointer notice if the pointer does not match a detected KB.
   - `(none)` if neither pointer exists.
4. Print detected KBs, one per line, marking the active one with `*`.
5. If no KBs were detected, suggest setup.

## Switch

When a KB name is provided:

1. Trim the value.
2. Reject values containing `/`, starting with `.` or `~`, or empty after trimming.
3. Check `<project-root>/<value>/wiki-config.json`. If missing, list detected KBs and stop.
4. Read the JSON and confirm a `collection` field exists.
5. Write `.agents/active-kb` with the trimmed basename and a trailing newline.
6. Confirm:

```text
Active KB set to '<value>' (<absolute-path>).
QMD collection: <collection-from-json>.
```

Do not run QMD commands and do not update `.claude/active-kb`.
