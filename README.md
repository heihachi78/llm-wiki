# LLM Wiki

A Claude Code implementation of the **LLM Wiki** pattern — a persistent, compounding knowledge base built and maintained by an LLM.

Based on the original idea by [Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## What is this?

Instead of traditional RAG that rediscovers knowledge from scratch on every query, the LLM **incrementally builds and maintains a structured wiki** — interlinked markdown files that sit between you and your raw source documents. The wiki compounds over time: cross-references are already there, contradictions are flagged, and synthesis reflects everything you've read.

You never write the wiki yourself. You curate sources, ask questions, and direct the analysis. The LLM does the summarizing, cross-referencing, and bookkeeping.

## Setup

1. Clone this repo into your project directory
2. Open it with [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
3. Drop source documents (markdown, PDF) into `raw/` and start with `/ingest`

The `raw/` and `wiki/` directories are created automatically on first use.

## Commands

| Command | Description |
|---------|-------------|
| `/ingest <path>` | Process a source document — creates a summary page, updates entities, concepts, cross-references, and the overview |
| `/query <question>` | Research a question using wiki knowledge + active web research, then update the wiki with discoveries |
| `/wiki <question>` | Search the wiki only (no web research) — fast internal lookup |
| `/lint` | Health-check the wiki for broken links, contradictions, orphan pages, missing cross-references |

## How it works

**Three layers:**

- **Raw sources** (`raw/`) — your curated source documents. Immutable — the LLM reads but never modifies them.
- **The wiki** (`wiki/`) — LLM-generated markdown pages: source summaries, entity pages, concept pages, analyses, an overview, and an index. The LLM owns this entirely.
- **The schema** (`CLAUDE.md`) — tells the LLM how the wiki is structured, what conventions to follow, and how to operate. You and the LLM co-evolve this over time.

## Browsing the wiki

The wiki is designed to work with [Obsidian](https://obsidian.md/) — open the `wiki/` folder as a vault to get graph view, backlinks, and live navigation. It also works as plain markdown files in any editor.

## Credits

Original concept: [Andrej Karpathy — LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
