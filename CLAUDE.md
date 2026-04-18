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
  commands/          # Slash-command prompts — source of truth for /setup, /ingest, /query, /wiki, /lint, /bug
  skills/qmd/        # QMD search skill wrapper
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

Commands locate the KB by globbing the project for `wiki-config.json` using patterns `*/wiki-config.json` and `*/*/wiki-config.json`. The file's **parent directory is the KB root**; the file contents are just `{ "collection": "<name>" }`. If no config is found, commands default to `knowledge-base/` in the project root with `collection = wiki` and suggest running `/setup`.

Multiple KBs can coexist in the project as sibling directories, each with its own `wiki-config.json`. Commands will find whichever the user intends via context (arguments, recent activity) or prompt if ambiguous.

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

- `/setup <path>` — Configure KB root and initialise QMD collection. Run once; re-run safely after renames.
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
