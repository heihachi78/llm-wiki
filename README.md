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

## Using with Codex

This repo is Codex-ready through the local `llm-wiki` skill:

```text
$llm-wiki
```

In Codex, `$llm-wiki` is a **skill invocation**. It is not a separate agent, not a shell command, and not the old Claude slash-command system. The skill loads the wiki-specific operating rules from `.agents/skills/llm-wiki/SKILL.md`, then routes your request to the appropriate workflow reference under `.agents/skills/llm-wiki/references/`.

You can invoke it explicitly:

```text
$llm-wiki wiki: how does data leakage relate to ML pipelines?
```

or in a more natural form:

```text
Use $llm-wiki to answer from wiki only: how does data leakage relate to ML pipelines?
```

The word after `$llm-wiki` selects the workflow:

- `setup:` configures a KB folder and QMD collection.
- `use:` switches or inspects the active KB.
- `ingest:` processes raw source files into the wiki.
- `wiki:` answers from existing wiki pages only, with no web research.
- `query:` answers using wiki + raw-source rereading + web research when needed, then folds useful discoveries back into the wiki.
- `lint:` audits wiki health.
- `bug:` logs workflow issues to `bugs.md`.

Examples:

```text
$llm-wiki use
$llm-wiki use: machine-learning
$llm-wiki ingest: machine-learning/raw/supervised/15_mistakes.pdf
$llm-wiki wiki: explain the role of stratified cross-validation
$llm-wiki query: what are current best practices for class imbalance?
$llm-wiki lint
$llm-wiki bug
```

The important distinction is:

- `wiki:` is a **closed-book wiki lookup** over existing wiki content. It should not use web research or add new outside facts.
- `query:` is a **research workflow**. It starts from the wiki, re-reads relevant raw sources because wiki summaries are lossy, uses web research when the question needs it, and updates the wiki with properly attributed findings.

## Codex Workflows

The old Claude slash commands are now represented by the `llm-wiki` Codex skill and its workflow references.

| Old Claude command | Codex workflow prompt | Description |
|---|---|---|
| `/setup <path>` | `$llm-wiki setup: <path>` | Configure the KB root folder, initialize the QMD search index when available, and set this KB as active |
| `/use` | `$llm-wiki use` | Show the active KB and all detected KBs |
| `/use <name>` | `$llm-wiki use: <name>` | Set `.agents/active-kb` to a configured KB basename |
| `/ingest [path]` | `$llm-wiki ingest: [path]` | Process a source document — creates a summary page, updates entities, concepts, cross-references, and the overview |
| `/wiki <question>` | `$llm-wiki wiki: <question>` | Search the wiki only — no web research |
| `/query <question>` | `$llm-wiki query: <question>` | Research a question using wiki knowledge + raw-source rereading + active web research, then update the wiki with discoveries |
| `/lint` | `$llm-wiki lint` | Health-check the wiki for broken links, contradictions, orphan pages, missing cross-references |
| `/bug` | `$llm-wiki bug` | Review the current session for wiki workflow issues and log them to `bugs.md` |

### Active KB selection

Codex uses `.agents/active-kb` as the per-clone active-KB pointer. It contains a single basename such as:

```text
machine-learning
```

The basename must match a sibling KB folder that contains `wiki-config.json`. If `.agents/active-kb` is missing, Codex may read `.claude/active-kb` as a read-only legacy fallback during migration, but Codex workflows write only `.agents/active-kb`.

If multiple KBs are detected and no active pointer is set, Codex lists the detected KBs and asks which basename to use for that run.

### QMD behavior in Codex

Codex workflows use your existing `qmd` installation and its existing QMD index. They do not install, rebuild, replace, or relocate QMD unless you explicitly ask.

If `qmd` comes from an nvm-managed Node installation, Codex should run it with that same `bin` directory at the front of `PATH` so the wrapper uses the matching Node runtime:

```bash
PATH="$(dirname "$(which qmd)"):$PATH" qmd status
```

The same prefix can be applied to `qmd query`, `qmd vsearch`, `qmd update`, and `qmd embed` when needed.

If QMD is unavailable or broken, Codex should continue with file-based wiki work and report `[QMD not available - semantic steps skipped]` once.

### Legacy Claude files

The old `.claude/commands/*.md` prompt files are kept as migration reference material. The Codex source of truth is:

```text
.agents/skills/llm-wiki/SKILL.md
.agents/skills/llm-wiki/references/*.md
AGENTS.md
```

## How it works

**Five layers:**

- **Raw sources** (`<kb-root>/raw/`) — your curated source documents. Immutable — the LLM reads but never modifies them.
- **The wiki** (`<kb-root>/wiki/`) — LLM-generated markdown pages: source summaries, entity pages, concept pages, analyses, an overview, and an index. The LLM owns this entirely.
- **The search index** (QMD) — a local semantic index over `<kb-root>/wiki/`. Workflows query it to find relevant pages before reading them, so operations stay fast as the wiki grows.
- **The schema** (`AGENTS.md`) — tells Codex how the wiki is structured, what conventions to follow, and how to operate. You and the LLM co-evolve this over time.
- **The workflow skill** (`.agents/skills/llm-wiki/`) — Codex-specific operational procedures for setup, ingest, query, wiki-only lookup, lint, and bug reporting.

## Browsing the wiki

The wiki is designed to work with [Obsidian](https://obsidian.md/) — open the `<kb-root>/wiki/` folder as a vault to get graph view, backlinks, and live navigation. It also works as plain markdown files in any editor.

## Credits

Original concept: [Andrej Karpathy — LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
