# Multi-KB Routing Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Eliminate ambiguity when multiple knowledge bases coexist by introducing an active-KB pointer (`.claude/active-kb`), a `/use` command, and a single shared resolution rule referenced from CLAUDE.md.

**Architecture:** This repo's "code" is markdown slash-command prompts in `.claude/commands/`. We add one new command (`/use`), one pointer file, and update every existing command's preamble to reference a single canonical rule in CLAUDE.md. `/setup` writes the pointer; `/ingest` detects cross-KB writes and asks the operator how to proceed.

**Tech Stack:** Markdown prompt files, Glob/Read/Write/Bash tools used by the agent at runtime, QMD CLI (unchanged). No automated tests — verification is manual against the live repo, which already contains multiple KBs (`term-int-foly-inf/` is the configured one; `felho-prog/`, `reinforcement-learning-wiki/`, `knowledge-base/` are listed in `.gitignore`).

**Spec:** `docs/superpowers/specs/2026-05-02-multi-kb-routing-design.md` — read it before starting.

**Source-fidelity note:** The plan asks you to write or modify markdown prompt content. Where exact text is provided below, copy it verbatim — these prompts are what the agent will literally execute.

---

## File map

**Create:**
- `.claude/commands/use.md` — new slash command (Task 3)

**Modify:**
- `CLAUDE.md` — replace the "KB root resolution" section with the canonical rule (Task 1)
- `.gitignore` — add `.claude/active-kb` (Task 2)
- `.claude/commands/setup.md` — add Step 10: write the pointer (Task 4)
- `.claude/commands/ingest.md` — replace preamble + add cross-KB detection (Task 5, Task 6)
- `.claude/commands/query.md` — replace preamble (Task 5)
- `.claude/commands/wiki.md` — replace preamble (Task 5)
- `.claude/commands/lint.md` — replace preamble (Task 5)
- `README.md` — add `/use` row + multi-KB workflow note (Task 7)

**Out of scope (do not touch):**
- `.claude/commands/bug.md` — does not read KB state
- Anything inside `<kb-root>/wiki/` — content unchanged
- `.claude/skills/qmd/` — wrapper unchanged

---

## Task 1: Update CLAUDE.md with the canonical resolution rule

**Files:**
- Modify: `CLAUDE.md` (lines 36-40, the current "KB root resolution" section)

**Why first:** Every other change references this section. Land it before changing any command preamble.

- [ ] **Step 1: Read the current section to confirm exact line range**

Read `CLAUDE.md` lines 36-40. The current block is:

```
## KB root resolution (all commands)

Commands locate the KB by globbing the project for `wiki-config.json` using patterns `*/wiki-config.json` and `*/*/wiki-config.json`. The file's **parent directory is the KB root**; the file contents are just `{ "collection": "<name>" }`. If no config is found, commands default to `knowledge-base/` in the project root with `collection = wiki` and suggest running `/setup`.

Multiple KBs can coexist in the project as sibling directories, each with its own `wiki-config.json`. Commands will find whichever the user intends via context (arguments, recent activity) or prompt if ambiguous.
```

- [ ] **Step 2: Replace the section verbatim with the canonical rule**

Use Edit to replace the entire block above with the following text:

````markdown
## KB root resolution (all commands)

The KB a command operates on is determined by `.claude/active-kb` — a single-line file containing the basename of the active KB folder (e.g. `term-int-foly-inf`). The basename is also the QMD collection name; they are 1:1.

The pointer is local to each clone (gitignored — see `.gitignore`). It is written by `/use <name>` and by `/setup <path>`. No other command writes it.

Every command that operates on a KB starts with this **resolution rule** — this section is the canonical version; commands reference it by name rather than duplicating it:

1. **Read `.claude/active-kb`** if it exists. Trim the contents and treat them as `<kb-name>`. If `<kb-name>` is empty, contains a slash, or starts with `.` or `~`, print `.claude/active-kb should contain a single basename like 'term-int-foly-inf'; got '<value>'. Run /use <name> to fix.` and stop. Otherwise, if `<project-root>/<kb-name>/wiki-config.json` exists, set `<root>` to that directory and read `<collection>` from the JSON. Done.
2. **Stale pointer.** If `.claude/active-kb` exists and is well-formed but `<project-root>/<kb-name>/` or its `wiki-config.json` is gone, print a one-line notice (`Active-KB pointer is stale: <kb-name> not found.`) and continue to step 3.
3. **Auto-detect.** Glob `*/wiki-config.json` and `*/*/wiki-config.json` from the project root:
   - **Zero hits** — suggest `/setup <path>`. Stop.
   - **Exactly one hit** — use it silently. (No prompt; the single-KB case must remain frictionless.)
   - **Multiple hits** — print a numbered list of detected KBs (basename + absolute path) and ask the operator: "Reply with the basename to use for this run, or run `/use <name>` to set it persistently." When the operator replies with a basename, treat it as a one-shot selection: resolve `<root>` and `<collection>` against that KB and continue, but do **not** write `.claude/active-kb`. The pointer is only written by `/use` and `/setup`.
4. **QMD availability.** Check `which qmd > /dev/null 2>&1`. If unavailable, skip QMD steps in the calling command and note `[QMD not available — semantic steps skipped]` once.

Pointer inspection and switching:
- `/use` (with no arguments) is the canonical way to inspect the current pointer. `/use <name>` is the canonical way to switch.
````

- [ ] **Step 3: Verify the edit**

Run: `grep -n "## KB root resolution" CLAUDE.md`
Expected: a single match at the new location, followed (when reading) by the new 4-step rule.

Run: `grep -n "active-kb" CLAUDE.md`
Expected: at least 4 matches (the description, gitignore reference, write rules, malformed-message).

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "Make KB root resolution rule canonical in CLAUDE.md

Replaces the loose 'commands will find whichever the user intends'
paragraph with a 4-step rule that every command preamble will
reference. Introduces the .claude/active-kb pointer."
```

---

## Task 2: Gitignore the active-KB pointer

**Files:**
- Modify: `.gitignore` (the `# Claude Code internal` block at lines 10-13)

**Why second:** Land the ignore before any command can write the pointer, so accidental commits never happen.

- [ ] **Step 1: Add the ignore entry**

Use Edit on `.gitignore` to add `.claude/active-kb` to the existing `# Claude Code internal` block. Replace:

```
# Claude Code internal
.claude/settings.json
.claude/plans/
.claude/memory/
```

with:

```
# Claude Code internal
.claude/settings.json
.claude/plans/
.claude/memory/
.claude/active-kb
```

- [ ] **Step 2: Verify**

Run: `grep -n "active-kb" .gitignore`
Expected: one match: `.claude/active-kb`.

- [ ] **Step 3: Commit**

```bash
git add .gitignore
git commit -m "Gitignore .claude/active-kb (per-clone pointer)"
```

---

## Task 3: Create the `/use` command

**Files:**
- Create: `.claude/commands/use.md`

- [ ] **Step 1: Write the new command file**

Use Write to create `.claude/commands/use.md` with this exact content:

````markdown
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
````

- [ ] **Step 2: Verify the file lands**

Run: `ls -la .claude/commands/use.md && head -3 .claude/commands/use.md`
Expected: file exists, first line is `# Use Knowledge Base: $ARGUMENTS`.

- [ ] **Step 3: Manual verification**

Open Claude Code in this repo and run, in this order:

1. `/use` (no args) → expect a status block listing `term-int-foly-inf` (and any other KBs the operator has cloned), with `(none — pointer file ...)` as the active line because no pointer exists yet on most clones.
2. `/use does-not-exist` → expect `KB 'does-not-exist' not found. Detected KBs: ...`. Verify `.claude/active-kb` was **not** created (`ls .claude/active-kb` should error if it didn't exist before).
3. `/use term-int-foly-inf` → expect confirmation. Verify `.claude/active-kb` now contains exactly `term-int-foly-inf` plus a newline (`cat .claude/active-kb`).
4. `/use` (no args, again) → expect `Active KB: term-int-foly-inf (...)` and `*` marker on that row.
5. `/use ../some/path` → expect rejection (`/use accepts a KB folder basename, not a path.`).

Pointer file (`.claude/active-kb`) is gitignored, so it will not appear in `git status`.

- [ ] **Step 4: Commit**

```bash
git add .claude/commands/use.md
git commit -m "Add /use command for active-KB inspection and switching

Two modes: bare /use prints status (active KB + detected KBs);
/use <name> validates the target KB exists and writes the
.claude/active-kb pointer. Never touches wiki or QMD state."
```

---

## Task 4: Make `/setup` write the active-KB pointer

**Files:**
- Modify: `.claude/commands/setup.md` (currently 9 steps, ending at line 65 "Setup complete...")

- [ ] **Step 1: Insert a new Step 9 (write pointer) and renumber the existing Step 9 to Step 10**

Open `.claude/commands/setup.md`. Find the existing block:

```
### Step 9: Confirm
Tell the user: "Setup complete. All wiki commands will now use `<root>/`."
```

Replace it with:

```
### Step 9: Set as active KB
Write `.claude/active-kb` (relative to the project root) with the value `<collection>` (the basename derived in Step 1) followed by a single trailing newline. Overwrite if it exists. This makes the just-configured KB the active one for subsequent commands. The pointer is gitignored — it does not appear in `git status`.

### Step 10: Confirm
Tell the user: "Setup complete. All wiki commands will now use `<root>/`. Active KB set to `<collection>`."
```

- [ ] **Step 2: Verify**

Run: `grep -n "^### Step" .claude/commands/setup.md`
Expected: 10 numbered steps (1 through 10), ending with `### Step 10: Confirm`.

Run: `grep -n "active-kb" .claude/commands/setup.md`
Expected: one match in Step 9.

- [ ] **Step 3: Manual verification**

In Claude Code: run `/setup term-int-foly-inf` (re-running is documented as safe). Confirm the agent walks through Steps 1-10 and that `cat .claude/active-kb` shows `term-int-foly-inf`.

- [ ] **Step 4: Commit**

```bash
git add .claude/commands/setup.md
git commit -m "/setup writes .claude/active-kb after configuring a KB

Setting up a KB is an explicit 'I'm working on this' signal, so the
pointer follows. New Step 9 writes the basename; old Step 9 becomes
Step 10 (confirmation) with an updated message."
```

---

## Task 5: Replace KB-resolution preambles in /ingest, /query, /wiki, /lint

**Files (all four get the same replacement):**
- Modify: `.claude/commands/ingest.md`
- Modify: `.claude/commands/query.md`
- Modify: `.claude/commands/wiki.md`
- Modify: `.claude/commands/lint.md`

**Why bundled:** Each preamble is currently identical text (the same 3 numbered points). They all become the same one-paragraph reference. Combining keeps the diffs aligned and the commit clean.

- [ ] **Step 1: Identify the exact block in each file**

For each of the four command files, locate the preamble block. It looks like this in each (with command-appropriate filename in the comment, but the body is identical):

```
### Preamble: Resolve KB root and QMD availability

1. Discover the wiki config by searching the project for a file named `wiki-config.json` — use Glob with patterns `*/wiki-config.json` and `*/*/wiki-config.json` from the project root. If found, set `root` to the directory containing that file (the config's own parent directory) and read `collection` from its JSON contents. If not found, default to `root = <project root>/knowledge-base/` and `collection = wiki`, and suggest the user run `/setup <path>`.
2. Use `<root>/` everywhere in this command instead of `knowledge-base/`.
3. Check QMD availability:
   ```bash
   which qmd > /dev/null 2>&1
   ```
   If QMD is not available, skip all QMD steps and note `[QMD not available — semantic steps skipped]` once in your response.
```

Verify with: `grep -n "### Preamble: Resolve KB root" .claude/commands/{ingest,query,wiki,lint}.md`
Expected: one match per file.

- [ ] **Step 2: Replace the block in each file with a reference to CLAUDE.md**

For each of the four files, use Edit to replace the entire preamble block above with this exact text:

```
### Preamble: Resolve KB root and QMD availability

Follow the **KB resolution rule** in `CLAUDE.md` (section "KB root resolution"). It defines steps 1-4 (read `.claude/active-kb`; handle stale pointer; auto-detect via glob with zero / one / many cases; check QMD availability). The result of those steps is `<root>` (KB folder), `<collection>` (QMD collection name), and a boolean for QMD availability.

Use `<root>/` everywhere in this command. If QMD is not available, skip all QMD steps and note `[QMD not available — semantic steps skipped]` once in your response.
```

This must be done for all four files: `ingest.md`, `query.md`, `wiki.md`, `lint.md`.

- [ ] **Step 3: Verify**

Run: `grep -c "Follow the \*\*KB resolution rule\*\*" .claude/commands/{ingest,query,wiki,lint}.md`
Expected output is one line per file in the form `<path>:<count>`, all showing `:1`. Example:

```
.claude/commands/ingest.md:1
.claude/commands/query.md:1
.claude/commands/wiki.md:1
.claude/commands/lint.md:1
```

Run: `grep -c "knowledge-base/" .claude/commands/{ingest,query,wiki,lint}.md`
Expected: every line shows `:0` (the old hard-coded default is gone — it now lives in CLAUDE.md's resolution rule and the suggestion is "run /setup").

Run: `grep -c "Glob with patterns" .claude/commands/{ingest,query,wiki,lint}.md`
Expected: every line shows `:0` (the duplicated rule text is gone).

- [ ] **Step 4: Manual verification**

In Claude Code, with `.claude/active-kb` set to `term-int-foly-inf`, run `/wiki "test query"`. Expect: the agent reads the pointer, uses `term-int-foly-inf/` silently (no prompt), and proceeds with the search. Then delete the pointer (`rm .claude/active-kb`) and run `/wiki "test query"` again. If the operator only has `term-int-foly-inf/` cloned, expect silent fall-through (one hit). If multiple KBs are present, expect the numbered-list prompt.

- [ ] **Step 5: Commit**

```bash
git add .claude/commands/ingest.md .claude/commands/query.md .claude/commands/wiki.md .claude/commands/lint.md
git commit -m "Point command preambles at the canonical KB resolution rule

ingest, query, wiki, lint all referenced the same duplicated 3-step
preamble. Replace each with a one-paragraph reference to CLAUDE.md's
KB resolution rule. Single source of truth; no drift between commands."
```

---

## Task 6: Add cross-KB detection to `/ingest`

**Files:**
- Modify: `.claude/commands/ingest.md` (insert a new step between the preamble and Step 0)

**Why separate from Task 5:** Task 5 is a pure refactor (same behavior, less duplication). Task 6 is a new behavioral guard. Keeping them in separate commits makes the diff history clear.

- [ ] **Step 1: Insert the cross-KB check**

Open `.claude/commands/ingest.md`. After the new (refactored) preamble and before `### Step 0: Resolve source (if no argument given)`, insert this new block:

```
### Preamble (continued): Cross-KB check

If `$ARGUMENTS` is non-empty (i.e. the operator gave a path), determine which KB owns the path:

1. Resolve `$ARGUMENTS` to an absolute path.
2. Walk up from that path's directory until a `wiki-config.json` is found, or until you reach the project root.
3. If a `wiki-config.json` is found, the basename of its containing directory is `<source-kb>`. Otherwise (no ancestor wiki-config) `<source-kb>` is undefined.

Then:

- **`<source-kb>` undefined or equal to `<collection>`** — proceed with the active KB. (Common case: path is inside `<root>/raw/`, or path is unrelated to any KB and the operator wants it ingested into the active one.)
- **`<source-kb>` is a different KB** — do **not** proceed silently. Print:
  ```
  This source lives in `<source-kb>`, but the active KB is `<collection>`.
  Choose:
    1. Switch active KB to `<source-kb>` and ingest there.
    2. Ingest into `<source-kb>` just this once (do not change the active pointer).
    3. Cancel.
  ```
  Wait for the operator's reply, then act:
  - On `1`: write `<source-kb>` to `.claude/active-kb` (overwrite), re-resolve `<root>` and `<collection>` against `<source-kb>`, continue with Step 0+.
  - On `2`: re-resolve `<root>` and `<collection>` against `<source-kb>` for this run only; do **not** modify `.claude/active-kb`. Continue with Step 0+.
  - On `3`: stop. Make no file changes.

Skip this block entirely if `$ARGUMENTS` is empty (Step 0 will handle source listing within the active KB).
```

- [ ] **Step 2: Verify**

Run: `grep -n "^### " .claude/commands/ingest.md`
Expected: a list of headings, with `### Preamble: Resolve KB root and QMD availability` followed by `### Preamble (continued): Cross-KB check` followed by `### Step 0: Resolve source (if no argument given)` and so on through Step 9.

Run: `grep -n "Cross-KB" .claude/commands/ingest.md`
Expected: one match.

- [ ] **Step 3: Manual verification**

Pre-conditions: `.claude/active-kb` contains `term-int-foly-inf`. The operator has at least one other KB cloned (e.g. `felho-prog/`) with a real raw file inside it. For Task 6's verification (testing the prompt + cancel path) an empty placeholder is fine: `touch felho-prog/raw/__cross-kb-test.md`.

In Claude Code, run `/ingest felho-prog/raw/__cross-kb-test.md`. Expect the disambiguation prompt with three numbered options. Reply `3` (cancel) — confirm no wiki pages were created in either KB and `.claude/active-kb` is unchanged.

Then run `/ingest term-int-foly-inf/raw/<some-existing-already-ingested-file>`. Expect no prompt (silent fast path because `<source-kb>` matches `<collection>`).

Clean up: `rm felho-prog/raw/__cross-kb-test.md` if it was fabricated.

(Scenario 10 in Task 8 also exercises the cross-KB prompt — the "switch and ingest" branch — and that one needs a *real* raw file with content, not a placeholder, since reply `1` would proceed to summarize it. Use a real file in `felho-prog/raw/` for that scenario.)

- [ ] **Step 4: Commit**

```bash
git add .claude/commands/ingest.md
git commit -m "/ingest detects cross-KB writes and asks before proceeding

Walks up from the source path to find its owning wiki-config.json. If
the path lives in a different KB than the active one, presents three
options (switch + ingest, one-shot ingest, cancel) instead of silently
writing pages into the wrong KB. Common case (path inside active KB)
stays silent."
```

---

## Task 7: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add `/use` to the commands table**

Use Edit to insert a new row in the commands table (between the existing `/setup` and `/ingest` rows). Replace:

```
| `/setup <path>` | Configure the KB root folder and initialize the QMD search index — run once before first use |
| `/ingest [path]` | Process a source document — creates a summary page, updates entities, concepts, cross-references, and the overview. Omit the path to list unprocessed files in `raw/` |
```

with:

```
| `/setup <path>` | Configure the KB root folder, initialize the QMD search index, and set this KB as active — run once per KB before first use |
| `/use [name]` | Switch the active KB to `<name>` (basename of a configured KB), or with no argument print the current active KB and the list of detected ones |
| `/ingest [path]` | Process a source document — creates a summary page, updates entities, concepts, cross-references, and the overview. Omit the path to list unprocessed files in `raw/` |
```

- [ ] **Step 2: Replace the multi-KB paragraph**

Replace the existing line:

```
Multiple KBs can coexist as sibling folders in the project — each with its own `wiki-config.json`. Commands auto-discover them and disambiguate from context.
```

with:

```
Multiple KBs can coexist as sibling folders in the project — each with its own `wiki-config.json`. Run `/setup <path>` to configure a new one (which also makes it active). Switch between configured KBs with `/use <name>`. Inspect the current active KB and the full list with `/use` (no argument). The active selection is stored in `.claude/active-kb` (gitignored, per-clone). When multiple KBs are configured and no pointer is set, commands prompt you to pick one for that run.
```

- [ ] **Step 3: Verify**

Run: `grep -n "/use" README.md`
Expected: at least 3 matches (table row + multi-KB paragraph references).

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "Document /use command and multi-KB workflow in README"
```

---

## Task 8: Final integration verification

**Files:** none (verification only)

This task does not modify code. It exercises the end-to-end flow against the live repo.

- [ ] **Step 1: Set up scratch state**

Pre-conditions: working tree clean (`git status` shows no changes after Task 7's commit). Operator has at least `term-int-foly-inf/` cloned. If a second KB is available (e.g. `felho-prog/`), use it for cross-KB tests; otherwise note the cross-KB test as "deferred until a second KB is cloned" and skip steps that need it.

- [ ] **Step 2: Run the spec's manual verification scenarios**

For each scenario, execute the command in Claude Code and confirm the expected outcome. (These mirror the "Testing" section of the spec.)

1. **Status display, no pointer.** `rm -f .claude/active-kb`. Run `/use`. Expect: `Active KB: (none ...)` and the list of detected KBs.
2. **Set pointer.** Run `/use term-int-foly-inf`. Expect confirmation. Verify `cat .claude/active-kb` prints `term-int-foly-inf`.
3. **Status display, pointer set.** Run `/use`. Expect `Active KB: term-int-foly-inf (...)` with `*` on that row.
4. **Silent fast path.** Run `/wiki "anything that exists in the wiki"`. Expect no KB-selection prompt.
5. **Bad name.** Run `/use does-not-exist`. Expect rejection listing valid KBs. Verify `cat .claude/active-kb` still shows `term-int-foly-inf` (unchanged).
6. **Path argument.** Run `/use ../foo`. Expect rejection (slashes / leading `.`). Pointer unchanged.
7. **Stale pointer recovery.** `echo "no-such-kb" > .claude/active-kb`. Run `/wiki "test"`. Expect a stale notice followed by auto-detection (silent if one KB, prompt if many). Re-run `/use term-int-foly-inf` to restore.
8. **`/setup` activates.** Run `/setup term-int-foly-inf` (re-run is safe). Expect `cat .claude/active-kb` reads `term-int-foly-inf`.
9. **Cross-KB ingest (if second KB available).** Activate `term-int-foly-inf`. Run `/ingest <path-into-felho-prog>` (use any real file inside `felho-prog/raw/` or fabricate one). Expect the 3-option prompt. Reply `3`; confirm nothing was written.
10. **Cross-KB ingest, switch case.** Repeat scenario 9 but reply `1`, using a *real* raw file with content (not the empty placeholder from Task 6). Expect: `.claude/active-kb` now contains `felho-prog`, ingest proceeds (real summary page is written, QMD re-indexes `felho-prog`). **Side effects:** scenario 10 writes wiki pages into `felho-prog/wiki/` and updates that KB's QMD collection — both effects live entirely inside the `felho-prog/` folder, which is its own separate git repo (per CLAUDE.md). Revert via that KB's git tree if you want a clean rollback (e.g. `cd felho-prog && git restore wiki/`), or keep the result if the ingest was useful. Restore the original active KB afterward by running `/use term-int-foly-inf`.

- [ ] **Step 3: Restore working state**

Re-set the operator's preferred active KB (`/use term-int-foly-inf`). Remove any scratch raw files created during scenario 9-10. Run `git status`; expect a clean tree.

- [ ] **Step 4: Verification report**

Write a short summary in the chat: "Manual verification complete. Scenarios 1-10 passed (or: scenarios X, Y deferred because second KB not present)." No commit — this task is verification only.

---

## Done

After Task 8, the implementation is complete. The repo's command set now resolves the active KB through a single canonical rule, exposes `/use` for inspection and switching, and prevents silent cross-KB writes in `/ingest`.

If any verification step in Task 8 reveals a gap, that's a real bug — return to the relevant earlier task, fix, re-verify, and re-commit before declaring done.
