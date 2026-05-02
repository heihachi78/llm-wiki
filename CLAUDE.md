# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This repo is an implementation of the **LLM Wiki** pattern: a persistent, compounding research knowledge base that Claude Code builds and maintains. The "code" here is almost entirely slash-command prompts — each command is a markdown file in `.claude/commands/` that drives an agent workflow. The user curates sources, asks questions, and directs analysis; the LLM owns summarizing, cross-referencing, and bookkeeping.

Two roles to keep distinct:
- **This repo (code)** — slash commands, skill wrappers, conventions. Tracked in git.
- **The KB (data)** — one or more knowledge-base folders living alongside this repo, each with `raw/` (immutable sources) and `wiki/` (LLM-generated pages). Gitignored by default; each KB is typically its own separate git repo.

## Repository layout

```
.claude/
  commands/          # Slash-command prompts — source of truth for /setup, /use, /ingest, /query, /wiki, /lint, /bug
  skills/            # Skill wrappers used by commands (qmd for semantic search, plus obsidian-*, defuddle, json-canvas helpers)
  active-kb          # Single-line basename pointer to the active KB (gitignored, per-clone; written only by /setup and /use)
  settings.local.json  # Allowlist for qmd CLI permissions
CLAUDE.md            # This file — wiki schema + conventions (read by every command)
README.md            # User-facing docs
bugs.md              # Operator-logged issues from /bug sessions — read before editing commands
<kb-root>/           # One per knowledge base; folder name is the QMD collection name
  wiki-config.json   # { "collection": "<name>" } — its location defines the KB root
  raw/               # Source documents (immutable)
  wiki/              # LLM-generated pages
```

## Editing commands

Commands are prompt files, not scripts. To change behavior, edit the relevant file in `.claude/commands/`:
- Steps are numbered and followed literally by the agent — keep ordering explicit.
- Every command starts by discovering the KB root (see below) before doing work. Don't skip that preamble when adding new commands.
- Before changing a command, check `bugs.md` — past fixes are recorded there and the commands encode those lessons.

## KB root resolution (all commands)

The KB a command operates on is determined by `.claude/active-kb` — a single-line file containing the basename of the active KB folder (e.g. `term-int-foly-inf`). The basename is also the QMD collection name; they are 1:1.

The pointer is local to each clone (gitignored — see `.gitignore`). It is written by `/use <name>` and by `/setup <path>`. No other command writes it.

Every command that operates on a KB starts with this **resolution rule** — this section is the canonical version; commands reference it by name rather than duplicating it:

1. **Read `.claude/active-kb`** if it exists. Trim the contents and treat them as `<kb-name>`. If `<kb-name>` is empty, contains a slash, or starts with `.` or `~`, print `.claude/active-kb should contain a single basename like 'term-int-foly-inf'; got '<value>'. Run /use <name> to fix.` and stop. Otherwise, if `<project-root>/<kb-name>/wiki-config.json` exists, set `<root>` to that directory and read `<collection>` from the JSON. Done.
2. **Stale pointer.** If `.claude/active-kb` exists and is well-formed but `<project-root>/<kb-name>/` or its `wiki-config.json` is gone, print a one-line notice (`Active-KB pointer is stale: <kb-name> not found.`) and continue to step 3.
3. **Auto-detect.** Glob `*/wiki-config.json` and `*/*/wiki-config.json` from the project root:
   - **Zero hits** — suggest `/setup <path>`. Stop.
   - **Exactly one hit** — use it silently. (No prompt; the single-KB case must remain frictionless.)
   - **Multiple hits** — print a numbered list of detected KBs (basename + absolute path) and ask the operator: "Reply with the basename to use for this run, or run `/use <name>` to set it persistently." When the operator replies with a basename, treat it as a one-shot selection: resolve `<root>` and `<collection>` against that KB and continue, but do **not** write `.claude/active-kb`. The pointer is only written by `/use` and `/setup`.
4. **QMD availability.** Check `which qmd > /dev/null 2>&1`. If unavailable, skip QMD steps in the calling command and note `[QMD not available — semantic steps skipped]` once.

Pointer inspection and switching:
- `/use` (with no arguments) is the canonical way to inspect the current pointer. `/use <name>` is the canonical way to switch.

## External dependency: QMD

`/ingest`, `/query`, `/wiki`, `/lint`, and `/setup` call the [QMD](https://github.com/tobilu/qmd) (`@tobilu/qmd`) CLI for local semantic search over `<kb-root>/wiki/`. Install once:

```bash
npm install -g @tobilu/qmd   # or: bun install -g @tobilu/qmd
```

Commands check availability with `which qmd` and degrade gracefully (skip semantic steps, note once in the response) if missing. When adding a step that uses QMD, follow the same pattern — don't hard-fail.

Key QMD operations used by commands:
- `qmd collection add <root>/wiki --name <collection>` — register (once per KB)
- `qmd context add qmd://<collection> "<description>"` — describe for retrieval
- `qmd update -c <collection>` then `qmd embed` — re-index after wiki changes (`qmd embed` has no `-c` flag, it embeds all collections)
- `qmd vsearch "<term>" -c <collection> -n 3 --json` — dedupe check before creating entity/concept pages
- `qmd status` — verify

## Wiki page schema

Every wiki page has YAML frontmatter and a markdown body:

```yaml
---
title: "Page Title"
type: concept        # concept | entity | source | analysis | overview
tags: [tag1, tag2]
sources: [source-filename.md]
date_created: YYYY-MM-DD
date_updated: YYYY-MM-DD
---
```

Section structure by page type:
- **Source:** Summary, Key Findings, Key Quotes, Methodology (if applicable), Relevance, Contradictions, Source Info
- **Entity:** Summary, Details, Related Work, Sources
- **Concept:** Summary, Details, Open Questions, Sources
- **Analysis:** Summary, Findings, Methodology, Sources

The `<kb-root>/wiki/` tree:
```
index.md       # Content catalog — read first on any operation
log.md         # Chronological operation log (append-only)
overview.md    # High-level synthesis / evolving thesis
sources/       # One page per ingested source
entities/      # People, organizations, tools, datasets
concepts/      # Ideas, theories, methods, frameworks
analyses/      # Comparisons, syntheses, filed query outputs
```

## Source fidelity (read this before writing any wiki page)

The wiki must reflect the source material, not the LLM's prior knowledge. Hallucinated quotes and invented details are the single biggest failure mode. Treat the following as hard rules:

- **Quotes are verbatim.** A `> "..."` block must be a literal substring of the source — copy-paste, not reconstruction. If you cannot copy it directly from the source you read, do not present it as a quote. Paraphrase in plain prose instead, with no quotation marks.
- **Every factual claim attributed to a source must be in that source.** Names, numbers, dates, definitions, methodology details, conclusions — if it is not in the source you read, do not write it. Do not "fill in" plausible context from background knowledge.
- **Stay close to the source's own wording.** Tight paraphrase or direct quotation is preferred over reformulation. When in doubt, quote verbatim — even copying a full sentence is better than introducing drift via synonyms, generalizations, or stylistic rewrites.
- **No invented entities or concepts.** Only create entity/concept pages for things actually named or described in the source. Do not extrapolate to "related" people, tools, or ideas the source does not mention.
- **Honest empty sections.** If a section (Key Quotes, Methodology, Contradictions, etc.) has no material in the source, write `None in source.` (or equivalent). Do not pad sections to look complete.
- **Uncertainty must be visible.** If you are not sure a claim is in the source, either re-read to verify, or omit it. Never guess.
- **Web research is the only exception.** During `/query`, information from web searches *is* allowed to introduce new claims — but each such claim must be attributed to its external URL/source on the page where it lands, not silently merged into existing source-derived content. Wiki content originating from `<root>/raw/` and wiki content originating from web research must remain distinguishable (e.g. `According to <url>, ...`).
- **When integrating into existing pages,** apply the same rule: a new source can only justify additions that are actually in that new source. Do not "tidy up" or rewrite existing claims using the new source as cover.

If a step in a command would naturally produce content not present in the source (e.g. "give 3-5 quotes" but the source yields only 1), produce only what the source supports and say so. Under-filling is correct; fabricating is not.

## Conventions

- **Filenames:** kebab-case (`transformer-architecture.md`).
- **Links:** relative paths with `.md` extensions (Obsidian + plain markdown compatible). From `concepts/` → `[Entity](../entities/entity.md)`; same directory → `[Other](other.md)`.
- **Index entries:** `- [Title](path/to/page.md) — One-line summary`.
- **Log entries:** `## [YYYY-MM-DD] operation | Title` followed by a brief description.
- **Cross-references:** always bidirectional. When you add or update a page, check reciprocal links on the other side.
- **Tables inside list items:** don't — Obsidian doesn't render indented tables. Use bullet lists instead (see `bugs.md` Issue 1).

## When updating existing pages

- Integrate rather than overwrite. Preserve prior content and extend it.
- Update `date_updated`; append the new source to the `sources` list.
- Flag contradictions explicitly: `Note: [new source] contradicts [existing claim] — [explanation]`.
- Update cross-references on both sides of any new link.

## Available slash commands

- `/setup <path>` — Configure KB root and initialise QMD collection. Run once; re-run safely after renames. Also sets the new KB as active.
- `/use [name]` — Switch the active KB (`.claude/active-kb`) to `<name>`, or with no argument print the current pointer and the list of detected KBs. Canonical way to inspect or change the active KB.
- `/ingest [path]` — Process a source into the wiki. With no argument, lists `<root>/raw/` cross-referenced against `index.md` and prompts.
- `/query <question>` — Answer using wiki + active web research, then fold discoveries back into the wiki.
- `/wiki <question>` — Wiki-only lookup; no web research.
- `/lint` — Health-check: broken links, contradictions, orphans, missing cross-references. When fixing malformed `sources:` fields, never leave them as `[]` — resolve to the correct source page filename (see `bugs.md` Issue 2).
- `/bug` — Log issues observed in the current session to `bugs.md`.

## Workflow guardrails

- Always read `<kb-root>/wiki/index.md` before any operation to understand current state.
- After any wiki-modifying operation, update both `<kb-root>/wiki/index.md` and `<kb-root>/wiki/log.md`.
- Never modify files in `<kb-root>/raw/` — sources are immutable.
- When judgment calls about emphasis or interpretation come up, ask the user rather than guessing.
- The wiki is read by humans (often via Obsidian), not just by retrieval — write accordingly.
