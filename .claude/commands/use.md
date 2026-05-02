# Use Knowledge Base: $ARGUMENTS

You are the **active-KB selector** for the wiki. Your job is to set or display which knowledge base subsequent commands operate on. The pointer lives in `.claude/active-kb` (gitignored, per-clone) and contains a single line: the KB folder basename (which is also the QMD collection name).

This command does **not** modify wiki content, QMD indexes, or any KB folder. It only reads or writes `.claude/active-kb`.

## Instructions

Read `CLAUDE.md` first to understand the KB resolution rule and the role of the pointer.

### Mode A: `$ARGUMENTS` is empty — read-only status

1. Detect KBs by globbing the project root for `*/wiki-config.json` and `*/*/wiki-config.json`. For each hit, the parent directory's basename is the KB name; the absolute path is the parent directory.
2. Read `.claude/active-kb` if it exists. Treat its contents (trimmed) as `<active-name>`.
3. Print a status block:
   - If the pointer exists and its `<active-name>` matches one of the detected KBs: `Active KB: <active-name> (<absolute-path>)`.
   - If the pointer exists but `<active-name>` is not in the detected list (stale): `Active KB: <active-name> [STALE — folder not found]`.
   - If the pointer is missing: `Active KB: (none — pointer file .claude/active-kb does not exist)`.
4. Print the list of detected KBs, one per line, with `*` marking the active one and `-` for the others. Format:
   ```
   * term-int-foly-inf   /Users/.../term-int-foly-inf
   - felho-prog          /Users/.../felho-prog
   ```
5. If zero KBs were detected, say: `No KBs detected. Run /setup <path> to configure one.`
6. Stop. Do not modify any file.

### Mode B: `$ARGUMENTS` is a name — set the pointer

1. Trim `$ARGUMENTS`. Reject input that contains a slash (`/`), starts with `.` or `~`, or is empty after trimming. On reject, print: `/use accepts a KB folder basename, not a path. Got: '<value>'.` and stop.
2. Check that `<project-root>/<value>/wiki-config.json` exists. If not, print `KB '<value>' not found. Detected KBs:` followed by the same listing as Mode A step 4 (without the active marker — there's no valid active yet from this call's perspective). Stop. **Do not write `.claude/active-kb`.**
3. Read the JSON at `<project-root>/<value>/wiki-config.json` and confirm it has a `collection` field (the value is informational; do not rename anything).
4. Write `.claude/active-kb` with the trimmed `<value>` followed by a single trailing newline. Use the Write tool. Overwrite if it already exists.
5. Print confirmation:
   ```
   Active KB set to '<value>' (<absolute-path>).
   QMD collection: <collection-from-json>.
   ```
6. Stop. Do not run `qmd update`, `qmd embed`, or modify any wiki content — `/use` is purely a router-state change.

### Important

- This command never writes inside any KB folder, never modifies `wiki-config.json`, and never touches QMD state.
- Mode A is read-only; Mode B writes exactly one file (`.claude/active-kb`) and only after validating the target KB exists.
- The pointer is gitignored. It is per-clone state.
- To configure a new KB, use `/setup <path>` (which also activates it). To switch between already-configured KBs, use `/use <name>`. To inspect, use `/use` with no arguments.
