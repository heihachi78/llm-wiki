# Setup Wiki: $ARGUMENTS

You are the **setup agent** for a research wiki. Your job is to configure the KB root folder and initialise the QMD search index so all wiki commands can find the right location and use semantic search.

## Instructions

### Step 1: Resolve the path
`$ARGUMENTS` should be a path to the KB root folder (the folder containing `raw/` and `wiki/` subdirectories).

- Resolve it to an absolute path.
- Derive `<collection>` as the folder's basename (e.g. `knowledge-base`, `pemik-wiki`).
- The basename must be URI-safe (letters, digits, hyphens, underscores). If it isn't, warn the user and suggest a safe name.

### Step 2: Write config
Write `<root>/wiki-config.json` **inside the KB folder itself**:
```json
{
  "collection": "<basename>"
}
```
The file's own location defines the KB root, so no `root` field is needed. Make sure `<root>/` exists before writing (Step 3 creates the standard subdirectories if missing).

### Step 3: Create or verify structure
Ensure the KB tree exists. From the project root:

```bash
mkdir -p <root>/raw
mkdir -p <root>/wiki/sources
mkdir -p <root>/wiki/entities
mkdir -p <root>/wiki/concepts
mkdir -p <root>/wiki/analyses
```

`mkdir -p` is idempotent — it creates missing directories and leaves existing ones untouched. If `<root>` itself is a relative path, resolve it against the project root first. If `<root>` does not exist as a folder yet, create it the same way.

### Step 3.5: Seed wiki scaffolding (only if absent)

For each of the three top-level wiki files below, check whether the file exists. If it does, leave it untouched. If it does not, create it with the seed content shown. This step must be idempotent — re-running `/setup` against a populated KB must not overwrite any existing content.

**`<root>/wiki/index.md`** — only if it does not already exist:

```markdown
# Wiki Index

Content catalog for this knowledge base. Every page belongs in exactly one section below. Each entry: `- [Title](path/to/page.md) — One-line summary`.

## Sources

_No sources ingested yet. Run `/ingest` to add one._

## Entities

_No entity pages yet._

## Concepts

_No concept pages yet._

## Analyses

_No analysis pages yet._
```

**`<root>/wiki/overview.md`** — only if it does not already exist:

```markdown
---
title: "Wiki Overview"
type: overview
date_created: <YYYY-MM-DD>
date_updated: <YYYY-MM-DD>
---

# Wiki Overview

High-level synthesis of this knowledge base. Updated by `/ingest` and `/query` when new material changes the big picture.

## Key Themes

_None yet — this section will populate as sources are ingested._

## Current Focus

_No active focus declared yet._

## Open Questions

_None yet._

## Gaps

_None identified yet._
```

Substitute today's date (UTC) for both `<YYYY-MM-DD>` placeholders.

**`<root>/wiki/log.md`** — only if it does not already exist:

```markdown
# Wiki Operation Log

Chronological, append-only log of operations against this wiki.
```

(Step 8 below will append the first dated entry to this file.)

### Step 4: Derive context description
- If `<root>/wiki/overview.md` exists: read it and summarise the wiki's domain in 1–2 sentences for use as QMD context.
- If it does not exist: use `"<collection> research knowledge base"` as the description.

### Step 5: Register QMD collection
First check whether the collection already exists:
```bash
qmd collection list
```
- If `<collection>` appears in the output: skip `qmd collection add` (already registered).
- If not found: run `qmd collection add <root>/wiki --name <collection>`

Then register context (always — this updates the description if it changed):
```bash
qmd context add qmd://<collection> "<description from Step 4>"
```

### Step 6: Index and embed
```bash
qmd update -c <collection>
qmd embed
```
Note: `qmd embed` has no `-c` flag — it embeds all collections. In a single-KB setup this is fine.

### Step 7: Verify
```bash
qmd status
```
Report the result to the user. Do not assert a specific document count — just show the status output.

### Step 8: Update log
Append to `<root>/wiki/log.md`:
```
## [YYYY-MM-DD] setup | Configured collection <collection>
KB root set to <absolute-path>. QMD collection initialised.
```

### Step 9: Set as active KB
Write `.claude/active-kb` (relative to the project root) with the value `<collection>` (the basename derived in Step 1) followed by a single trailing newline. Overwrite if it exists. This makes the just-configured KB the active one for subsequent commands. The pointer is gitignored — it does not appear in `git status`.

### Step 10: Confirm
Tell the user: "Setup complete. All wiki commands will now use `<root>/`. Active KB set to `<collection>`."

## Important
- Re-running `/setup` is safe — it overwrites the config (inside the KB folder), updates the QMD context without duplicating the collection, and re-creates only those wiki scaffold files (`index.md`, `overview.md`, `log.md`, and the `wiki/{sources,entities,concepts,analyses}/` subdirectories) that are currently missing. Files and directories that already exist are left untouched.
- Run `/setup` again whenever you rename or move the KB folder.
- If QMD is not installed, Steps 5–7 will fail. Install it first: `npm install -g @tobilu/qmd` or `bun install -g @tobilu/qmd`.
