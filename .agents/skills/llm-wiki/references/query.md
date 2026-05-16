# Research Query Workflow

Use when the user asks to answer with wiki knowledge plus current or external research, and to fold discoveries back into the wiki.

1. Read `references/common.md`; resolve `<root>` and `<collection>`.
2. Read `<root>/wiki/index.md` and relevant wiki pages. If the wiki is empty, say research works best after ingesting sources, then continue only if the user still wants web research.
3. If QMD is available, run `qmd query "<question>" -c <collection> -n 10 --json` with escalated permissions, not in the sandbox.
4. Re-read relevant raw sources before web research:
   - Use `sources:` frontmatter on relevant wiki pages to find raw files under `<root>/raw/`.
   - Scan for details relevant to this question that the wiki summary may have missed.
   - Keep raw-derived findings separate from web-derived findings.
5. Do web research when the question needs current, external, or cross-check information. Use official/primary sources where possible. Cite URLs. Do not silently merge web claims into raw-derived content.
6. Use subagents only if the user explicitly asks for delegation, subagents, or parallel agent work. Otherwise do the research locally.
7. Synthesize the answer:
   - Lead with the direct answer.
   - Cite wiki pages, raw sources, and external URLs distinctly.
   - Flag contradictions.
   - State confidence and gaps.
8. Update the wiki:
   - Add raw-derived missing material to existing source/concept/entity pages or create supported new pages.
   - Add web-derived material only with explicit URL attribution.
   - Do not tidy, rewrite, or strengthen old claims unless the supporting source justifies it.
   - Add bidirectional cross-references.
   - Update `date_updated` on touched pages.
9. Update `<root>/wiki/overview.md` if the research changes the big picture.
10. Update `<root>/wiki/index.md` for new or significantly changed pages.
11. Append `<root>/wiki/log.md`:

```markdown
## [YYYY-MM-DD] query | Brief question summary
Researched and synthesized answer. Re-read N raw sources; created/updated M wiki pages.
Key finding: most important discovery in one sentence.
```

12. If QMD is available, run `qmd update -c <collection>` and `qmd embed` with escalated permissions, not in the sandbox.
13. Ask whether to save the synthesized answer as an analysis page. If yes, create it, cross-link it, update `index.md`, append `log.md`, and re-index if QMD is available.

Never fabricate quotes. Never use source-internal locators on concept/entity/analysis pages.
