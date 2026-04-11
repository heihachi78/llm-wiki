# Setup Wiki: $ARGUMENTS

You are the **setup agent** for a research wiki. Your job is to configure the KB root folder and initialise the QMD search index so all wiki commands can find the right location and use semantic search.

## Instructions

### Step 1: Resolve the path
`$ARGUMENTS` should be a path to the KB root folder (the folder containing `raw/` and `wiki/` subdirectories).

- Resolve it to an absolute path.
- Derive `<collection>` as the folder's basename (e.g. `knowledge-base`, `pemik-wiki`).
- The basename must be URI-safe (letters, digits, hyphens, underscores). If it isn't, warn the user and suggest a safe name.

### Step 2: Write config
Write `.claude/wiki-config.json` at the project root (next to `.claude/`):
```json
{
  "root": "<absolute-path>",
  "collection": "<basename>"
}
```

### Step 3: Verify structure
Check that `<root>/raw/` and `<root>/wiki/` both exist.
- If either is missing, warn the user and ask whether to continue anyway.

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

### Step 9: Confirm
Tell the user: "Setup complete. All wiki commands will now use `<root>/`."

## Important
- Re-running `/setup` is safe — it overwrites the config and updates the QMD context without duplicating the collection.
- Run `/setup` again whenever you rename or move the KB folder.
- If QMD is not installed, Steps 5–7 will fail. Install it first: `npm install -g @tobilu/qmd` or `bun install -g @tobilu/qmd`.
