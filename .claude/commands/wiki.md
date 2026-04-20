# Wiki Search: $ARGUMENTS

You are the **wiki search agent** for a research wiki. Your job is to answer questions using only the existing wiki content — no web research. For questions that need external information, suggest the user run `/query` instead.

## Instructions

Read CLAUDE.md first for wiki conventions, then follow this workflow step-by-step:

### Preamble: Resolve KB root and QMD availability

1. Discover the wiki config by searching the project for a file named `wiki-config.json` — use Glob with patterns `*/wiki-config.json` and `*/*/wiki-config.json` from the project root. If found, set `root` to the directory containing that file (the config's own parent directory) and read `collection` from its JSON contents. If not found, default to `root = <project root>/knowledge-base/` and `collection = wiki`, and suggest the user run `/setup <path>`.
2. Use `<root>/` everywhere in this command instead of `knowledge-base/`.
3. Check QMD availability:
   ```bash
   which qmd > /dev/null 2>&1
   ```
   If QMD is not available, skip all QMD steps and note `[QMD not available — semantic steps skipped]` once in your response.

### Step 1: Understand existing knowledge
Read `<root>/wiki/index.md` and `<root>/wiki/overview.md` to understand the wiki's scope, major themes, and all available pages. If the wiki is empty or the index has no entries, tell the user and suggest running `/ingest` on some sources first — there's nothing to search yet. The index tells you what pages exist; the overview tells you how they connect and what the key themes are.

Identify pages relevant to the question in `$ARGUMENTS` — check across all categories: sources, concepts, entities, and analyses.

**If QMD available:** additionally run:
```bash
qmd query "<question from $ARGUMENTS>" -c <collection> -n 10 --json
```
Union the pages from QMD results with those identified via index.md. Pages found only by QMD (not surfaced by index.md navigation) are included and flagged as "(surfaced via semantic search)" in the synthesis.

### Step 2: Read relevant pages
Read the most relevant wiki pages identified in Step 1. Follow cross-references to gather related context — a concept page may link to an entity that adds important detail, or a source page may contain evidence that supports a claim.

### Step 3: Synthesize answer
Combine what you found into a clear, well-structured answer:
- Lead with a direct answer to the question
- Support with evidence from wiki pages, citing them with relative links
- Note confidence level — is the wiki comprehensive on this topic, or are there gaps?
- If the wiki has contradictions on this topic, surface them explicitly
- If there are significant knowledge gaps, suggest the user run `/query $ARGUMENTS` for web-augmented research

Present the answer to the user.

### Step 4: Offer to file the analysis
If the synthesized answer represents a useful new perspective or brings together information from multiple pages in a novel way, ask the user if it should be saved as a page in `<root>/wiki/analyses/`. If yes:
- Create the page with standard analysis format (frontmatter with type: analysis, Summary, Findings, Methodology, Sources)
- Add cross-references from the analysis to all relevant pages (and vice versa)
- Update `<root>/wiki/index.md` with the new analysis entry
- Append to `<root>/wiki/log.md`:
```
## [YYYY-MM-DD] wiki | Brief question summary
Searched wiki and synthesized answer. Created analysis page.
Key finding: most important insight in one sentence.
```

### Step 5: Update overview (if needed)
Read `<root>/wiki/overview.md`. If this search changes the big picture, clarifies an open question or contradiction, adds a new dimension to a major theme, challenges an existing thesis, or fills a significant gap — update the overview accordingly.
It's not mandatory to update the overview if the result of the search does not add real value (a new insight, perspective, etc.) to the existing overview or the synthesized information does not help to better understand the big picture.

### Important
- **Do not use WebSearch or WebFetch** — this command works exclusively from wiki content
- Do not create or update concept, entity, or source pages — this command only reads existing pages and optionally creates analysis pages
- When the wiki lacks sufficient information to answer well, be honest about it and recommend `/query`
- The goal is fast, wiki-internal lookup — not research
