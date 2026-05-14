# Common LLM Wiki Rules

Use this reference for every workflow in this skill.

## KB Root Resolution

The active KB pointer for Codex lives at `.agents/active-kb`. It contains one basename, which is also the QMD collection name. During migration, `.claude/active-kb` is a read-only legacy fallback.

1. Read `.agents/active-kb` if it exists. Trim the contents as `<kb-name>`.
2. If `.agents/active-kb` is missing and `.claude/active-kb` exists, read `.claude/active-kb` as a legacy fallback. Do not write `.claude/active-kb`.
3. Reject an empty value, a value containing `/`, or a value starting with `.` or `~`. Tell the user: `.agents/active-kb should contain a single basename like 'term-int-foly-inf'; got '<value>'. Use the active-KB workflow to fix it.`
4. If `<project-root>/<kb-name>/wiki-config.json` exists, set `<root>` to that directory and read `<collection>` from the JSON.
5. If the pointer is well-formed but stale, print `Active-KB pointer is stale: <kb-name> not found.` and auto-detect.
6. Auto-detect KBs by globbing `*/wiki-config.json` and `*/*/wiki-config.json` from the project root:
   - Zero hits: suggest setup and stop.
   - One hit: use it silently.
   - Multiple hits: list basename plus absolute path and ask the user which basename to use for this run. Do not write a pointer for one-shot selections.
7. Check `which qmd > /dev/null 2>&1`. If missing, set QMD availability to false.
8. If `qmd` exists but a QMD command fails because the installed package is broken, treat QMD as unavailable for that run and report the failure briefly.

## Required Context

- Read `AGENTS.md` before writing wiki content.
- Read `<root>/wiki/index.md` before any operation.
- Read `<root>/wiki/overview.md` when a workflow may affect the big-picture synthesis.
- Read `bugs.md` before editing workflow instructions or during lint/ingest/query when past bug behavior matters.

## QMD Commands

When QMD is available, use the user's existing `qmd` command and its existing index. Do not install, rebuild, reinstall, or replace QMD unless the user explicitly asks. Do not override `XDG_CACHE_HOME`, `XDG_CONFIG_HOME`, `HOME`, or QMD state paths unless the user explicitly asks.

If `which qmd` resolves inside an nvm version directory such as `/Users/.../.nvm/versions/node/<version>/bin/qmd`, run QMD with that same `bin` directory at the front of `PATH` so the wrapper uses its matching `node` binary:

```bash
PATH="$(dirname "$(which qmd)"):$PATH" qmd status
```

Apply the same `PATH=...` prefix to other QMD commands in that environment. This selects the existing QMD runtime; it does not change QMD's database or config.

```bash
qmd collection add <root>/wiki --name <collection>
qmd context add qmd://<collection> "<description>"
qmd update -c <collection>
qmd embed
qmd query "<question>" -c <collection> -n 10 --json
qmd vsearch "<term>" -c <collection> -n 3 --json
qmd status
```

`qmd embed` has no collection flag; it embeds all collections.

## Source Fidelity

- Quotes in `> "..."` blocks must be copied verbatim from a source actually read.
- Every source-derived factual claim must be present in the cited source.
- Do not create entity or concept pages for adjacent ideas that the source does not name or describe.
- If a section has no support in the source, write `None in source.` or a similarly honest empty value.
- Web research is allowed only in the query workflow. Attribute web-derived claims to their URL and keep them distinguishable from raw-derived wiki content.
