# LLM Wiki — Research Knowledge Base

This is a research wiki maintained by Claude Code. The wiki is a persistent, compounding knowledge base — structured, interlinked markdown files that sit between the user and raw source documents. The LLM builds and maintains all wiki pages. The user curates sources, asks questions, and directs the analysis.

## Setup

Run `/setup <path>` once before first use to configure the KB root folder and initialise the QMD search index. Re-run if you rename or move the KB folder.

The KB config lives at `<kb-root>/wiki-config.json` **inside the knowledge base folder itself**, not under `.claude/`. Commands locate it by searching the project for a file named `wiki-config.json` (glob `*/wiki-config.json` and `*/*/wiki-config.json` from the project root); the file's parent directory is the KB root, and the file holds just `{ "collection": "<name>" }`. If no config is found, commands default to `knowledge-base/` in the project root with `collection = wiki`.

## Directory Structure

```
<kb-root>/                # KB root folder — any name, configured via /setup
  raw/                    # Immutable source documents (never modify these)
    assets/               # Downloaded images referenced by sources
  wiki/                   # LLM-generated wiki pages (you own this entirely)
    index.md              # Content catalog — read this first for any operation
    log.md                # Chronological operation log (append-only)
    overview.md           # High-level synthesis and evolving thesis
    sources/              # One summary page per ingested source
    entities/             # People, organizations, tools, datasets
    concepts/             # Ideas, theories, methods, frameworks
    analyses/             # Comparisons, syntheses, filed query outputs
```

## Page Format

Every wiki page uses YAML frontmatter and markdown body:

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

### Section structure by page type

**Source pages:** Summary, Key Findings, Key Quotes, Methodology (if applicable), Relevance, Contradictions, Source Info
**Entity pages:** Summary, Details, Related Work, Sources
**Concept pages:** Summary, Details, Open Questions, Sources
**Analysis pages:** Summary, Findings, Methodology, Sources

## Conventions

- **Filenames:** kebab-case, e.g. `transformer-architecture.md`
- **Links:** Relative paths with `.md` extensions for Obsidian + plain markdown compatibility
  - From concepts/: `[Entity Name](../entities/entity-name.md)`
  - From sources/: `[Concept Name](../concepts/concept-name.md)`
  - Within same directory: `[Other Page](other-page.md)`
- **Index entries:** One line per page — `- [Title](path/to/page.md) — One-line summary`
- **Log entries:** `## [YYYY-MM-DD] operation | Title` followed by a brief description
- **Cross-references:** When creating or updating a page, always add links to related pages in other categories. When updating a page, check if other pages need reciprocal links.

## When updating existing pages

- Integrate new information into the existing structure — do not overwrite previous content
- Update `date_updated` in frontmatter
- Add new sources to the `sources` frontmatter list
- Flag contradictions explicitly: "Note: [new source] contradicts [existing claim] — [explanation]"
- Update cross-references on both sides of a new link

## Available Commands

- `/setup <path>` — Configure KB root folder and initialise QMD search index (run once before first use)
- `/ingest` — Process a new source document into the wiki
- `/query` — Ask a research question (with active web research)
- `/wiki` — Search the wiki for answers (wiki-only, no web research)
- `/lint` — Health-check the wiki for issues
- `/bug` — Review the session for wiki command issues and log them to `bugs.md`

## Workflow Tips

- Always read `<kb-root>/wiki/index.md` before any operation to understand current wiki state
- After any operation that modifies wiki pages, update both `<kb-root>/wiki/index.md` and `<kb-root>/wiki/log.md`
- When in doubt about emphasis or interpretation, ask the user
- The wiki should be browsable and useful on its own — write for a human reader, not just for retrieval
