# Lint Workflow

Use when the user asks to health-check a wiki.

1. Read `references/common.md`; resolve `<root>` and `<collection>`.
2. Read `<root>/wiki/index.md`, then systematically inspect wiki pages under sources, entities, concepts, and analyses.
3. Check high-severity issues:
   - Contradictions.
   - Stale claims superseded by newer sources.
   - Broken wiki links.
   - Broken or missing raw file references in source frontmatter or Source Info.
   - Malformed `sources:` values. When a matching source page exists, replace with the correct source page filename; never leave `sources: []` as a fix.
   - Unverifiable quote blocks against cited raw files.
   - Unsupported factual claims, spot-checking specific names, numbers, dates, definitions, and methodology details against cited sources.
4. Check medium-severity issues:
   - Stale or incomplete `overview.md`.
   - Orphan pages with no inbound links from content pages. Links from `index.md`, `log.md`, or `overview.md` do not count.
   - Missing concept/entity pages for important named items.
   - Weak cross-references, especially concept-analysis pairs sharing a topic or two or more tags.
   - Incomplete pages that can be filled from existing sources.
   - If QMD is available, run semantic gap checks with `qmd vsearch "<title> <summary>" -c <collection> -n 5 --json` using escalated permissions, not in the sandbox.
5. Check low-severity issues:
   - Formatting drift from `AGENTS.md`.
   - Missing frontmatter fields.
   - Index drift.
   - Suggested research gaps.
6. Present findings grouped by severity with affected pages and suggested fixes.
7. Ask which issues to fix. Do not fix lint findings without approval.
8. For approved fixes:
   - Apply source-fidelity and cross-reference rules.
   - Update `date_updated` on modified pages.
   - Update `index.md` if pages were added, removed, or substantially changed.
   - Append `log.md`:

```markdown
## [YYYY-MM-DD] lint | Fixed N issues
Brief summary of what was found and fixed.
```

9. If QMD is available and pages changed, run `qmd update -c <collection>` and `qmd embed` with escalated permissions, not in the sandbox.
