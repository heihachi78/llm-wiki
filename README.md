# LLM Wiki

A Claude Code implementation of the **LLM Wiki** pattern — a persistent, compounding knowledge base built and maintained by an LLM.

Based on the original idea by [Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## What is this?

Instead of traditional RAG that rediscovers knowledge from scratch on every query, the LLM **incrementally builds and maintains a structured wiki** — interlinked markdown files that sit between you and your raw source documents. The wiki compounds over time: cross-references are already there, contradictions are flagged, and synthesis reflects everything you've read.

You never write the wiki yourself. You curate sources, ask questions, and direct the analysis. The LLM does the summarizing, cross-referencing, and bookkeeping.

## Prerequisites

The wiki commands use [QMD](https://github.com/tobilu/qmd) (`@tobilu/qmd`) as a local semantic search index over your wiki pages. `/ingest`, `/query`, `/wiki`, and `/lint` all call the `qmd` CLI, so it must be installed globally before first use:

```bash
npm install -g @tobilu/qmd
# or
bun install -g @tobilu/qmd
```

Without QMD, the wiki commands will fail at their search steps.

## Setup

1. Install QMD (see [Prerequisites](#prerequisites)).
2. Clone this repo into your project directory.
3. Open it with [Claude Code](https://docs.anthropic.com/en/docs/claude-code).
4. Create your KB folder with `raw/` and `wiki/` subdirectories. The folder can be named anything — e.g. `knowledge-base/`, `my-research/`, `pemik-wiki/`.
5. Run `/setup <path-to-kb-folder>` once. This writes `wiki-config.json` inside the KB folder and registers the QMD collection.
6. Drop source documents (markdown, PDF) into `<kb-root>/raw/` and run `/ingest <filename>` (or just `/ingest` to list unprocessed files).

Re-running `/setup` is safe. Run it again whenever you rename or move the KB folder.

Multiple KBs can coexist as sibling folders in the project — each with its own `wiki-config.json`. Commands auto-discover them and disambiguate from context.

## Commands

| Command | Description |
|---------|-------------|
| `/setup <path>` | Configure the KB root folder and initialize the QMD search index — run once before first use |
| `/ingest [path]` | Process a source document — creates a summary page, updates entities, concepts, cross-references, and the overview. Omit the path to list unprocessed files in `raw/` |
| `/query <question>` | Research a question using wiki knowledge + active web research, then update the wiki with discoveries |
| `/wiki <question>` | Search the wiki only (no web research) — fast internal lookup |
| `/lint` | Health-check the wiki for broken links, contradictions, orphan pages, missing cross-references |
| `/bug` | Review the current session for wiki command issues and log them to `bugs.md` — commands read this file before editing, so fixes compound |

## How it works

**Four layers:**

- **Raw sources** (`<kb-root>/raw/`) — your curated source documents. Immutable — the LLM reads but never modifies them.
- **The wiki** (`<kb-root>/wiki/`) — LLM-generated markdown pages: source summaries, entity pages, concept pages, analyses, an overview, and an index. The LLM owns this entirely.
- **The search index** (QMD) — a local semantic index over `<kb-root>/wiki/`. Commands query it to find relevant pages before reading them, so operations stay fast as the wiki grows.
- **The schema** (`CLAUDE.md`) — tells the LLM how the wiki is structured, what conventions to follow, and how to operate. You and the LLM co-evolve this over time.

## Browsing the wiki

The wiki is designed to work with [Obsidian](https://obsidian.md/) — open the `<kb-root>/wiki/` folder as a vault to get graph view, backlinks, and live navigation. It also works as plain markdown files in any editor.

## Credits

Original concept: [Andrej Karpathy — LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
