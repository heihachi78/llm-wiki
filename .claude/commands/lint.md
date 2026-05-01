# Wiki Lint — Health Check

You are the **lint agent** for a research wiki. Your job is to audit the wiki for quality issues and fix them with user approval.

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

### Step 1: Scan the wiki
Read `<root>/wiki/index.md` to get a full picture of all pages. Then read through the wiki pages systematically — sources, entities, concepts, and analyses.

### Step 2: Check for issues
Audit every page for the following issues, organized by severity:

**High severity:**
- **Contradictions:** Pages that make conflicting claims about the same topic
- **Stale claims:** Information superseded by newer sources but not yet updated
- **Broken links:** Links to pages that don't exist
- **Broken raw file references:** Source pages whose `sources:` frontmatter or `Raw file:` path points to a file that doesn't exist under `<root>/raw/`. For each broken reference, search `<root>/raw/` subfolders for the correct filename and suggest the corrected path. Fix both the frontmatter `sources:` field and the `Raw file:` line in the Source Info section. When fixing a malformed or incorrect `sources:` frontmatter value (e.g. an external identifier or wrong path), scan `<root>/wiki/sources/` for the matching source page and insert its correct filename. Never leave `sources:` as `[]` if a corresponding source page already exists in the wiki.
- **Unverifiable quotes:** For every `> "..."` block on a page that cites a raw source (i.e. its `sources:` frontmatter points to a file under `<root>/raw/`), verify the quote is a literal substring of that raw file. Open the raw file and search for the exact quoted string (case-insensitive, ignoring whitespace differences is acceptable). If not found, flag it: the quote is fabricated or transformed. Suggested fix: locate the closest real passage, replace the quote with the verbatim text, or convert it to a plain-prose paraphrase (no quotation marks). For pages citing web sources, only check that an external URL is recorded — do not refetch the web.
- **Unsupported claims:** Pages whose body asserts specific facts (names, numbers, dates, definitions, methodology details) that are not present in any of the listed `sources:`. Spot-check by sampling 2-3 specific factual claims per page and locating them in the cited source(s). Flag claims that cannot be located. Suggested fix: remove the claim, soften it, or attach the correct source.

**Medium severity:**
- **Stale overview:** Does `<root>/wiki/overview.md` accurately reflect the wiki's current themes, scope, and gaps? Check: Are all major topic areas represented in Key Themes? Are listed gaps still actual gaps (or have they been filled)? Are there new topic clusters that the overview doesn't mention? Does the "Current Focus" still match what the wiki actually contains?
- **Orphan pages:** Pages with no inbound links from other *content pages* (sources, concepts, analyses, or entity pages). An entry in `index.md`, `log.md`, or `overview.md` does **not** count as an inbound link — the page is still an orphan if only those files reference it.
- **Missing pages:** Concepts or entities mentioned in text but lacking their own page
- **Weak cross-references:** Pages that clearly relate to each other but don't link. As a required step, explicitly compare every concept page against every analysis page: if they share a topic name or 2+ tags, they should link to each other. Concept/analysis pairs on the same subject (e.g. formal-theory concept + applied synthesis analysis) are the most common source of missed cross-references.
- **Incomplete pages:** Pages with empty or stub sections that could be filled from existing sources
- **Semantic gaps** *(only if QMD available):* For each wiki page, extract its title and one-line summary from `index.md`, then run:
  ```bash
  qmd vsearch "<title> <summary>" -c <collection> -n 5 --json
  ```
  From the results, take the top 3 by score (excluding the page itself). For each: check whether a link exists in either direction between the two pages. Flag as a semantic gap only if the candidate's score is in the top half of the returned result set (relative threshold — avoids flagging low-confidence matches). Present flagged pairs as: "Semantic gap: `page-a.md` ↔ `page-b.md` — highly similar but not linked."

**Low severity:**
- **Inconsistent formatting:** Pages that don't follow the standard format in CLAUDE.md
- **Missing frontmatter fields:** Pages with incomplete YAML frontmatter
- **Index drift:** Pages that exist but aren't listed in `<root>/wiki/index.md`, or index entries for pages that no longer exist
- **Suggested research:** Data gaps that could be filled with a web search

### Step 3: Present findings
Show the user a prioritized list of issues found, grouped by severity. For each issue:
- What the issue is
- Which pages are affected
- A suggested fix

### Step 4: Fix selected issues
Ask the user which issues to fix (they may want all, some, or none). For each selected issue:
- Make the fix following CLAUDE.md conventions
- Update cross-references as needed
- Update `date_updated` on modified pages

### Step 5: Update index and log
Update `<root>/wiki/index.md` if any pages were created, removed, or significantly changed.
Append to `<root>/wiki/log.md`:
```
## [YYYY-MM-DD] lint | Fixed N issues
Brief summary of what was found and fixed. E.g.: Fixed 2 broken links, added 3 missing cross-references, created 1 missing concept page.
```

### Important
- Be thorough — read every page, not just a sample
- Don't fix anything without user approval
- When suggesting research to fill gaps, be specific about what to search for
- The lint should leave the wiki healthier and more interconnected
