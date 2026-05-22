# Ingest Workflow

Use when the user asks to process a raw source into the wiki.

Hard rule: ingest never creates or updates files under `<root>/wiki/analyses/`, even if the source is a synthesis, overview, exam guide, or comparative document. Analysis pages are reserved for explicit user-requested wiki-level synthesis workflows, after searching the whole wiki and getting confirmation to file the result. During ingest, put source-supported synthesis material into the source page and, when appropriate, into entity/concept pages only.

1. Read `references/common.md`; resolve `<root>` and `<collection>`.
2. If the user provided a source path, resolve it absolutely and walk upward to see whether it belongs to a KB. If it belongs to a different KB than the active one, ask whether to switch, use that KB once, or cancel.
3. If no source path was provided:
   - List files under `<root>/raw/` recursively.
   - Read `<root>/wiki/index.md`.
   - Present which raw files appear to be already ingested and which are not.
   - Ask which file to ingest.
4. Read the source. For PDFs, use page ranges or extraction tooling as appropriate; for markdown/text, read the full file. Never modify `raw/`.
5. Present a short source briefing:
   - What the source covers.
   - 3-5 supported takeaways.
   - What seems most interesting.
   - How it relates to existing wiki pages from `index.md`.
6. Ask the user what to emphasize or de-emphasize before writing pages when the source admits multiple reasonable framings.
7. Create or update the source page in `<root>/wiki/sources/`:
   - Frontmatter: `title`, `type: source`, `tags`, `sources`, `date_created`, `date_updated`.
   - Sections: Summary, Key Findings, Key Quotes, Methodology if applicable, Relevance, Contradictions, Source Info, Detailed Content
   - Re-read relevant source passages while writing. Do not write from memory.
   - Include only verified verbatim quotes. If none are usable, say so.
   - The goal of the source page is NOT only to summarize the source: raw source files can be in any document format, and they usually structure the content differently. Thats why you need to create the source files: to preserve the relevant details and to create sources that are in the same format and structure.
8. Identify entities actually named or described in the source. Use QMD `vsearch` with escalated permissions when available before creating new pages. Update or create `<root>/wiki/entities/*.md` using only source-supported information.
9. Identify concepts actually discussed in the source. Use QMD `vsearch` with escalated permissions when available before creating new pages. Update or create `<root>/wiki/concepts/*.md` using only source-supported information.
10. Add bidirectional cross-references among all touched pages. The newly created source page must always reference the original raw source.
11. Update `<root>/wiki/overview.md` only if the source changes the big picture.
12. Update `<root>/wiki/index.md` entries for all new or substantially changed source, entity, and concept pages. Do not add analysis entries during ingest.
13. Append `<root>/wiki/log.md`:

```markdown
## [YYYY-MM-DD] ingest | Source Title
Brief description of what was ingested. N entity pages created/updated, M concept pages created/updated. 0 analysis pages created.
Key finding: most important takeaway in one sentence.
```

14. If QMD is available, run `qmd update -c <collection>` and `qmd embed` with escalated permissions, not in the sandbox.

Keep concept/entity pages free of source-internal locators. Attribute there by wiki-link to source pages.
