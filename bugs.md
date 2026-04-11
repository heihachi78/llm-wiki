# Bug Reports

## [2026-04-10] Bug Report

### Issue 1: Indented markdown tables don't render in Obsidian
- **Command:** /query
- **Severity:** Low
- **Problem:** When adding content inside a markdown list item (bullet point), a table was indented with spaces. Obsidian cannot render indented markdown tables, causing them to display as raw pipe-separated text.
- **Context:** The `/query` command expanded the Dijkstra entry in `graph-representations.md` with a complexity comparison table nested under a bullet point. Fixed by converting to a bullet list format instead.
- **Status:** Fixed (workaround: avoid tables inside list items; use bullet lists instead)

## [2026-04-10] Bug Report

### Issue 2: `/lint` leaves `sources:` fields empty after fixing malformed values
- **Command:** /lint
- **Severity:** Medium
- **Problem:** When fixing `sources:` frontmatter that contained external paper identifiers or raw file paths, the lint agent stripped the wrong values but set the field to `[]` rather than inserting the correct wiki source page filename(s). A second lint pass was required to fill in the correct values.
- **Context:** Triggered on `onion-construction-methods.md` and `custom-null-models-community-detection.md` — both existed at the time of the first lint run (2026-04-10, "post-resilience research" pass). The first lint logged fixing their `sources:` field but left it empty.
- **Status:** Open

### Issue 3: `/lint` did not detect orphan entity page
- **Command:** /lint
- **Severity:** Low
- **Problem:** `entities/ben-shepherd.md` had no inbound links from any content page (only `index.md`), but was not flagged as an orphan by the first lint pass even though the page existed at that time (created during the gravity model ingest preceding the first lint).
- **Context:** First lint pass (2026-04-10). Orphan detected and fixed by second lint pass in the same day.
- **Status:** Open

### Issue 4: `/lint` did not detect missing cross-reference between closely related pages
- **Command:** /lint
- **Severity:** Low
- **Problem:** `concepts/percolation-theory-network-resilience.md` and `analyses/network-resilience-analysis.md` are twin pages (formal theory vs. applied synthesis of the same topic) and did not link to each other. The first lint pass did not flag this weak cross-reference even though both pages existed at the time.
- **Context:** First lint pass (2026-04-10). Missing link detected and fixed by second lint pass.
- **Status:** Open

## [2026-04-11] Bug Report

### Issue 5: `/ingest` file listing doesn't indicate already-ingested sources
- **Command:** /ingest
- **Severity:** Low
- **Problem:** When invoked with no arguments, the agent listed all raw files without cross-referencing the wiki index to identify which had already been ingested. The user had to ask a follow-up question to get that information, requiring an extra round-trip.
- **Context:** `/ingest` called with no arguments; wiki already had 4 ingested sources at the time.
- **Status:** Open
