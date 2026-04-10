# Ingest Source: $ARGUMENTS

You are the **ingest agent** for a research wiki. Your job is to process a new source document and integrate its knowledge into the wiki.

## Instructions

Read CLAUDE.md first for wiki conventions, then follow this workflow:

### Step 1: Read the source
Read the file at the path provided in `$ARGUMENTS` (relative to the project root, typically in `raw/`). If it's a PDF, use the Read tool with page ranges. If it's markdown, read the full file.

### Step 2: Discuss key takeaways
Present the user with:
- A brief summary of what the source covers
- 3-5 key takeaways or findings
- What's most interesting or novel about this source
- How it relates to existing wiki content (read `wiki/index.md` to check)

Ask the user what to emphasize and whether anything should be highlighted or de-emphasized.

### Step 3: Create source summary page
Create a page in `wiki/sources/` with the standard source page format:
- Frontmatter: title, type: source, tags, sources (self-referencing the raw file), dates
- Sections: Summary, Key Findings, Methodology (if applicable), Relevance, Source Info
- Source Info should include: author(s), date, URL/DOI if available, file path to raw source

### Step 4: Identify and update entities
Scan the source for notable entities (people, organizations, tools, datasets, projects).
For each entity:
- If a page exists in `wiki/entities/`: update it with new information from this source, add the source to its sources list, update cross-references
- If no page exists: create a new entity page with the standard entity format

### Step 5: Identify and update concepts
Scan the source for key concepts (ideas, theories, methods, frameworks, techniques).
For each concept:
- If a page exists in `wiki/concepts/`: integrate new information, add source, update cross-references
- If no page exists: create a new concept page with the standard concept format

### Step 6: Update cross-references
Review all pages you created or modified. Ensure:
- Every page links to related pages in other categories
- Reciprocal links exist (if A links to B, B should link to A)
- The source page links to all entities and concepts it mentions

### Step 7: Update overview
Read `wiki/overview.md`. If this source changes the big picture — introduces a new major theme, challenges an existing thesis, or fills a significant gap — update the overview accordingly. If it's a minor addition, leave the overview as-is.

### Step 8: Update index
Add entries for all new pages to the appropriate sections in `wiki/index.md`. Update summaries of existing pages if they changed significantly. Each entry: `- [Title](path/to/page.md) — One-line summary`

### Step 9: Update log
Append an entry to `wiki/log.md`:
```
## [YYYY-MM-DD] ingest | Source Title
Brief description of what was ingested. N entity pages created/updated, M concept pages created/updated.
Key finding: most important takeaway in one sentence.
```

### Important
- Never modify files in `raw/` — sources are immutable
- When updating existing pages, integrate rather than overwrite
- Flag contradictions explicitly when new information conflicts with existing wiki content
- Ask the user before making judgment calls about emphasis or interpretation
- Update `date_updated` on every modified page
