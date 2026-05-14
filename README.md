# LLM Wiki

A Codex implementation of the **LLM Wiki** pattern — a persistent, compounding knowledge base built and maintained by an LLM.

Based on the original idea by [Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## What is this?

Instead of traditional RAG that rediscovers knowledge from scratch on every query, the LLM **incrementally builds and maintains a structured wiki** — interlinked markdown files that sit between you and your raw source documents. The wiki compounds over time: cross-references are already there, contradictions are flagged, and synthesis reflects everything you've read.

You never write the wiki yourself. You curate sources, ask questions, and direct the analysis. The LLM does the summarizing, cross-referencing, and bookkeeping.

## Prerequisites

The wiki workflows use [QMD](https://github.com/tobilu/qmd) (`@tobilu/qmd`) as a local semantic search index over your wiki pages. Ingest, query, wiki-only lookup, lint, and setup all call the `qmd` CLI when it is available:

```bash
npm install -g @tobilu/qmd
# or
bun install -g @tobilu/qmd
```

Without QMD, Codex can still do file-based wiki work, but semantic search and embedding refresh steps are skipped.

Codex workflows use your existing `qmd` installation and its existing index. They do not install, rebuild, replace, or override QMD cache/config paths unless you explicitly ask for that.

## Setup

1. Install QMD (see [Prerequisites](#prerequisites)).
2. Clone this repo into your project directory.
3. Open it with Codex.
4. Create your KB folder with `raw/` and `wiki/` subdirectories. The folder can be named anything — e.g. `knowledge-base/`, `my-research/`, `pemik-wiki/`.
5. Ask Codex: `Use $llm-wiki to setup <path-to-kb-folder>`. This writes `wiki-config.json` inside the KB folder, registers the QMD collection when possible, and sets the active KB.
6. Drop source documents (markdown, PDF) into `<kb-root>/raw/` and ask Codex: `Use $llm-wiki to ingest <filename>` or `Use $llm-wiki to list sources to ingest`.

Re-running setup is safe. Run it again whenever you rename or move the KB folder.

Multiple KBs can coexist as sibling folders in the project — each with its own `wiki-config.json`. Ask Codex to setup a new one, switch with `Use $llm-wiki to switch to <name>`, and inspect with `Use $llm-wiki to show the active KB`. The active selection is stored in `.agents/active-kb` (gitignored, per-clone). During migration, `.claude/active-kb` is read as a legacy fallback if the Codex pointer is missing.

## Codex Workflows

The old Claude slash commands are now represented by the `llm-wiki` Codex skill and its workflow references.

| Workflow prompt | Description |
|---------|-------------|
| `Use $llm-wiki to setup <path>` | Configure the KB root folder, initialize the QMD search index when available, and set this KB as active |
| `Use $llm-wiki to show/switch active KB` | Inspect or change `.agents/active-kb` |
| `Use $llm-wiki to ingest [path]` | Process a source document — creates a summary page, updates entities, concepts, cross-references, and the overview |
| `Use $llm-wiki to query <question>` | Research a question using wiki knowledge + active web research, then update the wiki with discoveries |
| `Use $llm-wiki to answer from wiki only: <question>` | Search the wiki only — no web research |
| `Use $llm-wiki to lint the active KB` | Health-check the wiki for broken links, contradictions, orphan pages, missing cross-references |
| `Use $llm-wiki to log wiki workflow bugs` | Review the current session for wiki workflow issues and log them to `bugs.md` |

## How it works

**Four layers:**

- **Raw sources** (`<kb-root>/raw/`) — your curated source documents. Immutable — the LLM reads but never modifies them.
- **The wiki** (`<kb-root>/wiki/`) — LLM-generated markdown pages: source summaries, entity pages, concept pages, analyses, an overview, and an index. The LLM owns this entirely.
- **The search index** (QMD) — a local semantic index over `<kb-root>/wiki/`. Workflows query it to find relevant pages before reading them, so operations stay fast as the wiki grows.
- **The schema** (`AGENTS.md`) — tells Codex how the wiki is structured, what conventions to follow, and how to operate. You and the LLM co-evolve this over time.
- **The workflow skill** (`.agents/skills/llm-wiki/`) — Codex-specific operational procedures for setup, ingest, query, wiki-only lookup, lint, and bug reporting.

## Browsing the wiki

The wiki is designed to work with [Obsidian](https://obsidian.md/) — open the `<kb-root>/wiki/` folder as a vault to get graph view, backlinks, and live navigation. It also works as plain markdown files in any editor.

## Credits

Original concept: [Andrej Karpathy — LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
