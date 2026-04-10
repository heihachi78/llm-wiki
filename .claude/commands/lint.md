# Wiki Lint — Health Check

You are the **lint agent** for a research wiki. Your job is to audit the wiki for quality issues and fix them with user approval.

## Instructions

Read CLAUDE.md first for wiki conventions, then follow this workflow step-by-step:

### Step 1: Scan the wiki
Read `wiki/index.md` to get a full picture of all pages. Then read through the wiki pages systematically — sources, entities, concepts, and analyses.

### Step 2: Check for issues
Audit every page for the following issues, organized by severity:

**High severity:**
- **Contradictions:** Pages that make conflicting claims about the same topic
- **Stale claims:** Information superseded by newer sources but not yet updated
- **Broken links:** Links to pages that don't exist
- **Broken raw file references:** Source pages whose `sources:` frontmatter or `Raw file:` path points to a file that doesn't exist under `raw/`. For each broken reference, search `raw/` subfolders for the correct filename and suggest the corrected path. Fix both the frontmatter `sources:` field and the `Raw file:` line in the Source Info section.

**Medium severity:**
- **Stale overview:** Does `wiki/overview.md` accurately reflect the wiki's current themes, scope, and gaps? Check: Are all major topic areas represented in Key Themes? Are listed gaps still actual gaps (or have they been filled)? Are there new topic clusters that the overview doesn't mention? Does the "Current Focus" still match what the wiki actually contains?
- **Orphan pages:** Pages with no inbound links from other pages
- **Missing pages:** Concepts or entities mentioned in text but lacking their own page
- **Weak cross-references:** Pages that clearly relate to each other but don't link
- **Incomplete pages:** Pages with empty or stub sections that could be filled from existing sources

**Low severity:**
- **Inconsistent formatting:** Pages that don't follow the standard format in CLAUDE.md
- **Missing frontmatter fields:** Pages with incomplete YAML frontmatter
- **Index drift:** Pages that exist but aren't listed in `wiki/index.md`, or index entries for pages that no longer exist
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
Update `wiki/index.md` if any pages were created, removed, or significantly changed.
Append to `wiki/log.md`:
```
## [YYYY-MM-DD] lint | Fixed N issues
Brief summary of what was found and fixed. E.g.: Fixed 2 broken links, added 3 missing cross-references, created 1 missing concept page.
```

### Important
- Be thorough — read every page, not just a sample
- Don't fix anything without user approval
- When suggesting research to fill gaps, be specific about what to search for
- The lint should leave the wiki healthier and more interconnected
