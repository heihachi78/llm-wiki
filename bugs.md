# Bug Reports

## [2026-04-10] Bug Report

### Issue 1: Indented markdown tables don't render in Obsidian
- **Command:** /query
- **Severity:** Low
- **Problem:** When adding content inside a markdown list item (bullet point), a table was indented with spaces. Obsidian cannot render indented markdown tables, causing them to display as raw pipe-separated text.
- **Context:** The `/query` command expanded the Dijkstra entry in `graph-representations.md` with a complexity comparison table nested under a bullet point. Fixed by converting to a bullet list format instead.
- **Status:** Fixed (workaround: avoid tables inside list items; use bullet lists instead)
