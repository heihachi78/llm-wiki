# Setup Workflow

Use when the user asks to configure or initialize a KB.

1. Resolve the requested path as the KB root. If relative, resolve it from the project root.
2. Derive `<collection>` from the KB folder basename. It must contain only letters, digits, hyphens, or underscores. If not, warn and suggest a safe rename.
3. Ensure the KB tree exists:
   - `<root>/raw`
   - `<root>/wiki/sources`
   - `<root>/wiki/entities`
   - `<root>/wiki/concepts`
   - `<root>/wiki/analyses`
4. Write `<root>/wiki-config.json`:

```json
{
  "collection": "<collection>"
}
```

5. Seed missing wiki scaffold files only if absent:
   - `<root>/wiki/index.md` with Sources, Entities, Concepts, and Analyses sections.
   - `<root>/wiki/overview.md` with `type: overview` frontmatter and today's date.
   - `<root>/wiki/log.md` with a `# Wiki Operation Log` heading.
6. Derive a 1-2 sentence QMD context from `overview.md`, or use `"<collection> research knowledge base"`.
7. If QMD is available:
   - Run `qmd collection list`.
   - Add the collection only if missing.
   - Run `qmd context add qmd://<collection> "<description>"`.
   - Run `qmd update -c <collection>` and `qmd embed`.
   - Run `qmd status` and summarize the status output.
8. Append `<root>/wiki/log.md`:

```markdown
## [YYYY-MM-DD] setup | Configured collection <collection>
KB root set to <absolute-path>. QMD collection initialised.
```

9. Write `.agents/active-kb` with `<collection>` and a trailing newline. Do not write `.claude/active-kb`.
10. Confirm the root and active KB. If QMD was unavailable, say `[QMD not available - semantic steps skipped]` once.

Re-running setup is safe: leave existing wiki files untouched, overwrite only `wiki-config.json`, refresh QMD if available, and update `.agents/active-kb`.
