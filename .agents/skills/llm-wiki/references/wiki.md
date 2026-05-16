# Wiki-Only Search Workflow

Use when the user asks to answer from the existing wiki only, with no web research.

1. Read `references/common.md`; resolve `<root>` and `<collection>`.
2. Read `<root>/wiki/index.md` and `<root>/wiki/overview.md`.
3. If the wiki is empty or has no relevant entries, say so and suggest ingesting sources or using the query workflow.
4. Identify relevant source, concept, entity, and analysis pages from the index.
5. If QMD is available, run `qmd query "<question>" -c <collection> -n 10 --json` with escalated permissions, not in the sandbox, and union high-relevance results with index-derived candidates.
6. Read the relevant wiki pages and follow useful cross-references.
7. Answer directly from wiki content only:
   - Cite wiki pages with relative links.
   - Quote only text actually present in the pages read.
   - Do not add prior knowledge or web information.
   - State gaps and confidence.
   - Surface contradictions if present.
8. If the synthesis is useful enough to preserve, ask whether to file it as `<root>/wiki/analyses/*.md`.
9. If the user approves filing:
   - Create an analysis page with frontmatter and Summary, Findings, Methodology, Sources sections.
   - Add bidirectional cross-references.
   - Update `index.md`.
   - Append `log.md`:

```markdown
## [YYYY-MM-DD] wiki | Brief question summary
Searched wiki and synthesized answer. Created analysis page.
Key finding: most important insight in one sentence.
```

10. Update `overview.md` only if the filed analysis changes the big-picture synthesis.
