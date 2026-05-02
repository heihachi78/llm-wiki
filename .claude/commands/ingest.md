# Ingest Source: $ARGUMENTS

You are the **ingest agent** for a research wiki. Your job is to process a new source document and integrate its knowledge into the wiki.

## Instructions

Read CLAUDE.md first for wiki conventions, then follow this workflow step-by-step:

### Preamble: Resolve KB root and QMD availability

Follow the **KB resolution rule** in `CLAUDE.md` (section "KB root resolution"). It defines steps 1-4 (read `.claude/active-kb`; handle stale pointer; auto-detect via glob with zero / one / many cases; check QMD availability). The result of those steps is `<root>` (KB folder), `<collection>` (QMD collection name), and a boolean for QMD availability.

Use `<root>/` everywhere in this command. If QMD is not available, skip all QMD steps and note `[QMD not available — semantic steps skipped]` once in your response.

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

### Step 0: Resolve source (if no argument given)
If `$ARGUMENTS` is empty or not provided, do the following before proceeding:
1. List all files under `<root>/raw/` recursively.
2. Read `<root>/wiki/index.md` to identify which raw files already have a corresponding source page (a source is ingested if it appears in the Sources section of the index).
3. Present a summary table:

| File | Status |
|------|--------|
| `<root>/raw/example.pdf` | ✓ ingested |
| `<root>/raw/other.pdf` | not yet ingested |

4. Ask the user which file to ingest next, then continue with Step 1 using the chosen file path.

### Step 1: Read the source
Read the file at the path provided in `$ARGUMENTS` (relative to the project root, typically in `<root>/raw/`). If it's a PDF, use the Read tool with page ranges. If it's markdown, read the full file.

### Step 2: Discuss key takeaways
Present the user with:
- A brief summary of what the source covers
- 3-5 key takeaways or findings
- What's most interesting or novel about this source
- How it relates to existing wiki content (read `<root>/wiki/index.md` to check)

Ask the user what to emphasize and whether anything should be highlighted or de-emphasized.

### Step 3: Create source summary page
Create a page in `<root>/wiki/sources/` with the standard source page format. **Re-read the relevant passages from the raw file as you write each section** — do not write from memory of Step 1. Apply the source-fidelity rules in CLAUDE.md throughout.

- Frontmatter: title, type: source, tags, sources (self-referencing the raw file), dates
- Sections: Summary, Key Findings, Key Quotes, Methodology (if applicable), Relevance, Contradictions, Source Info
- **Summary / Key Findings / Methodology / Relevance:** every claim must come from the source. Prefer the source's own phrasing over rewording. Do not introduce background context, definitions, or inferred conclusions that are not on the page you read.
- **Key Quotes:** up to 3-5 *verbatim* quotes from the source. Each `> "..."` must be a literal copy-paste from the raw file — same wording, same punctuation. **Verify each quote by locating it in the source before you write it.** If the source yields fewer than 3 strong verbatim quotes, include only what's actually there (even zero) and write `None in source.` or `Only N verbatim quote(s) found.` rather than padding with reconstructed or paraphrased "quotes". Never wrap a paraphrase in quotation marks.
- **Contradictions:** explicit section listing any conflicts with existing wiki content. If none, write "No contradictions identified." This gives contradiction tracking a permanent home rather than relying on inline flags.
- **Source Info:** author(s), date, URL/DOI if available, file path to raw source — only fields actually present in the source. If the author or date is not stated in the document, write `unknown` rather than guessing.

### Step 4: Identify and update entities
Scan the source for notable entities (people, organizations, tools, datasets, projects) **that are actually named or described in the source**. Do not extrapolate to related entities the source does not mention. For each entity:
- **If QMD available:** run `qmd vsearch "<entity name>" -c <collection> -n 3 --json` before deciding whether to create a new page. If a top result clearly matches the candidate entity, update that existing page rather than creating a duplicate.
- If a page exists in `<root>/wiki/entities/`: update it only with information actually present in this source, add the source to its sources list, update cross-references. Do not enrich the page with general knowledge about the entity that this particular source does not provide.
- If no page exists: create a new entity page. The page should describe the entity *as the source describes it* — not a Wikipedia-style summary from prior knowledge. If the source only mentions the entity in passing, the page can be brief; that's fine.

### Step 5: Identify and update concepts
Scan the source for key concepts (ideas, theories, methods, frameworks, techniques) **that the source actually discusses**. Do not invent concept pages for ideas merely *adjacent* to the source's topic. For each concept:
- **If QMD available:** run `qmd vsearch "<concept name>" -c <collection> -n 3 --json` before deciding whether to create a new page. If the top result looks like the same concept, update it rather than creating a duplicate.
- If a page exists in `<root>/wiki/concepts/`: integrate only what this source contributes (new definitions, examples, critiques, etc., that are in the source), add the source to the sources list, update cross-references. Do not rewrite existing claims using prior knowledge under cover of "integration".
- If no page exists: create a new concept page using the source's own framing and wording. Quote definitions verbatim where useful.

### Step 6: Update cross-references
Review all pages you created or modified. Ensure:
- Every page links to related pages in other categories
- Reciprocal links exist (if A links to B, B should link to A)
- The source page links to all entities and concepts it mentions

### QMD re-index (after Step 6, before Step 7)
If QMD available:
```bash
qmd update -c <collection>
qmd embed
```
Note: `qmd embed` has no `-c` flag — it embeds all collections.

### Step 7: Update overview
Read `<root>/wiki/overview.md`. If this source changes the big picture — introduces a new major theme, challenges an existing thesis, or fills a significant gap — update the overview accordingly. If it's a minor addition, leave the overview as-is.

### Step 8: Update index
Add entries for all new pages to the appropriate sections in `<root>/wiki/index.md`. Update summaries of existing pages if they changed significantly. Each entry: `- [Title](path/to/page.md) — One-line summary`

### Step 9: Update log
Append an entry to `<root>/wiki/log.md`:
```
## [YYYY-MM-DD] ingest | Source Title
Brief description of what was ingested. N entity pages created/updated, M concept pages created/updated.
Key finding: most important takeaway in one sentence.
```

### Important
- Never modify files in `<root>/raw/` — sources are immutable
- When updating existing pages, integrate rather than overwrite
- Flag contradictions explicitly when new information conflicts with existing wiki content
- Ask the user before making judgment calls about emphasis or interpretation
- Update `date_updated` on every modified page
