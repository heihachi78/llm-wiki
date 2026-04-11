# Ingest Source: $ARGUMENTS

You are the **ingest agent** for a research wiki. Your job is to process a new source document and integrate its knowledge into the wiki.

## Instructions

Read CLAUDE.md first for wiki conventions, then follow this workflow step-by-step:

### Step 0: Resolve source (if no argument given)
If `$ARGUMENTS` is empty or not provided, do the following before proceeding:
1. List all files under `knowledge-base/raw/` recursively.
2. Read `knowledge-base/wiki/index.md` to identify which raw files already have a corresponding source page (a source is ingested if it appears in the Sources section of the index).
3. Present a summary table:

| File | Status |
|------|--------|
| `knowledge-base/raw/example.pdf` | ✓ ingested |
| `knowledge-base/raw/other.pdf` | not yet ingested |

4. Ask the user which file to ingest next, then continue with Step 1 using the chosen file path.

### Step 1: Read the source
Read the file at the path provided in `$ARGUMENTS` (relative to the project root, typically in `knowledge-base/raw/`). If it's a PDF, use the Read tool with page ranges. If it's markdown, read the full file.

### Step 2: Discuss key takeaways
Present the user with:
- A brief summary of what the source covers
- 3-5 key takeaways or findings
- What's most interesting or novel about this source
- How it relates to existing wiki content (read `knowledge-base/wiki/index.md` to check)

Ask the user what to emphasize and whether anything should be highlighted or de-emphasized.

### Step 3: Create source summary page
Create a page in `knowledge-base/wiki/sources/` with the standard source page format:
- Frontmatter: title, type: source, tags, sources (self-referencing the raw file), dates
- Sections: Summary, Key Findings, Key Quotes, Methodology (if applicable), Relevance, Contradictions, Source Info
- Key Quotes: 3-5 notable direct quotes from the source, each with brief context (e.g. `> "Quote here" — context about why this matters`)
- Contradictions: explicit section listing any conflicts with existing wiki content. If none, write "No contradictions identified." This gives contradiction tracking a permanent home rather than relying on inline flags.
- Source Info should include: author(s), date, URL/DOI if available, file path to raw source

### Step 4: Identify and update entities
Scan the source for notable entities (people, organizations, tools, datasets, projects).
For each entity:
- If a page exists in `knowledge-base/wiki/entities/`: update it with new information from this source, add the source to its sources list, update cross-references
- If no page exists: create a new entity page with the standard entity format

### Step 5: Identify and update concepts
Scan the source for key concepts (ideas, theories, methods, frameworks, techniques).
For each concept:
- If a page exists in `knowledge-base/wiki/concepts/`: integrate new information, add source, update cross-references
- If no page exists: create a new concept page with the standard concept format

### Step 6: Update cross-references
Review all pages you created or modified. Ensure:
- Every page links to related pages in other categories
- Reciprocal links exist (if A links to B, B should link to A)
- The source page links to all entities and concepts it mentions

### Step 7: Update overview
Read `knowledge-base/wiki/overview.md`. If this source changes the big picture — introduces a new major theme, challenges an existing thesis, or fills a significant gap — update the overview accordingly. If it's a minor addition, leave the overview as-is.

### Step 8: Update index
Add entries for all new pages to the appropriate sections in `knowledge-base/wiki/index.md`. Update summaries of existing pages if they changed significantly. Each entry: `- [Title](path/to/page.md) — One-line summary`

### Step 9: Update log
Append an entry to `knowledge-base/wiki/log.md`:
```
## [YYYY-MM-DD] ingest | Source Title
Brief description of what was ingested. N entity pages created/updated, M concept pages created/updated.
Key finding: most important takeaway in one sentence.
```

### Important
- Never modify files in `knowledge-base/raw/` — sources are immutable
- When updating existing pages, integrate rather than overwrite
- Flag contradictions explicitly when new information conflicts with existing wiki content
- Ask the user before making judgment calls about emphasis or interpretation
- Update `date_updated` on every modified page
