---
name: llm-wiki
description: Operate and maintain this repository's LLM Wiki knowledge bases in Codex. Use when the user asks to setup or switch a KB, ingest raw sources, answer from the wiki, run web-augmented research, lint wiki health, or log wiki-command bugs. Trigger phrases include setup wiki, use KB, active KB, ingest source, wiki-only lookup, query the wiki, research query, lint wiki, and bug report for wiki workflows.
---

# LLM Wiki

Use this skill to maintain the repository's persistent research wikis: immutable `raw/` sources, LLM-written `wiki/` pages, QMD indexing, active-KB routing, source-fidelity rules, cross-links, `index.md`, and `log.md`.

## First Steps

1. Read the repository root `AGENTS.md` before any wiki operation. Treat it as the canonical schema and source-fidelity contract.
2. Read `bugs.md` before changing workflow instructions or when running lint/ingest/query workflows.
3. Resolve the active KB with the rule in `references/common.md`.
4. Always read `<root>/wiki/index.md` before answering from or modifying a KB.

## Workflow Router

- Setup or initialize a KB: read `references/setup.md`.
- Inspect or switch the active KB: read `references/use.md`.
- Ingest a raw source into wiki pages: read `references/ingest.md`.
- Answer using only existing wiki content: read `references/wiki.md`.
- Answer with wiki plus web research and fold discoveries back: read `references/query.md`.
- Audit wiki health and optionally fix issues: read `references/lint.md`.
- Log workflow bugs from the current session: read `references/bug.md`.

## Operating Rules

- Preserve existing user or KB changes. Integrate rather than overwrite.
- Never modify files in `<root>/raw/`.
- Source-derived wiki claims must be supported by the cited source. Verbatim quote blocks must be literal substrings of the source that was read.
- Source-internal locators such as slide, page, section, timestamp, or line numbers belong on source pages only. Concept, entity, and analysis pages cite by wiki-link to source pages.
- After any wiki-modifying operation, update `<root>/wiki/index.md`, append `<root>/wiki/log.md`, and re-index with QMD when available.
- QMD CLI commands must be run outside the sandbox with escalated permissions. Do not first try QMD in the sandbox: QMD depends on user-level sqlite/cache/model state outside the workspace and sandbox attempts are expected to fail.
- Treat QMD as unavailable only if `which qmd` is missing or an escalated QMD command fails for a real QMD/runtime reason. Do not label sandbox `SQLITE_CANTOPEN`, sqlite-vec, cache, model, or home-directory access errors as QMD unavailability.
