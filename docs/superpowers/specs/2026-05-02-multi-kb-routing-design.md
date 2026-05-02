# Multi-Knowledge-Base Routing — Design

**Date:** 2026-05-02
**Status:** Approved (pending spec review)

## Problem

This repo's slash commands operate on a knowledge base (KB) whose root is identified by a `wiki-config.json` file in the KB folder. Each command's preamble globs `*/wiki-config.json` and `*/*/wiki-config.json` from the project root and uses whatever it finds.

The user keeps several KBs as sibling directories (`term-int-foly-inf/`, `felho-prog/`, `reinforcement-learning-wiki/`, `knowledge-base/`, all gitignored). When more than one KB is present, the current preamble has no defined behavior — the agent silently picks one. CLAUDE.md claims commands "find whichever the user intends via context (arguments, recent activity) or prompt if ambiguous," but no command implements that.

The concrete failure mode: a command can write wiki pages into the wrong KB without warning. There is also no command-line way to see or change which KB is active.

## Scope

- A way to declare and switch the active KB.
- A single, shared resolution rule used by every command.
- Detection of the cross-KB-write hazard in `/ingest`.
- Documentation updates so command authors and operators agree on the model.

Out of scope:
- Changes to wiki page schema, QMD usage, or per-KB content.
- A `--kb=` flag on every command (not needed given the user's one-KB-per-session pattern; revisit only if usage shifts).
- Cross-KB queries that read from two KBs at once.

## Usage assumptions (from brainstorming)

- The operator works in one KB per session; switches between KBs are deliberate and infrequent.
- New KBs are added by running `/setup <path>`; that should be the natural moment to start working on them.
- A friendly disambiguation prompt is preferred over a hard error when the operator's intent is ambiguous.

## Design

### 1. Active-KB pointer

A new file `.claude/active-kb` (single line: the KB folder basename, e.g. `term-int-foly-inf`) is the single source of truth for which KB commands operate on.

- The basename is also the QMD collection name — they are already 1:1 in the existing design.
- Local, gitignored. The existing `.claude/` gitignore patterns already exclude `.claude/settings.json`, `.claude/plans/`, `.claude/memory/`; this design adds `.claude/active-kb` to that list. Operators may have different KBs cloned; the pointer is per-clone state.
- Written by `/setup` and `/use`. Read by every other command's preamble.
- If the file is missing or points to a folder that no longer contains `wiki-config.json`, the resolution rule (§3) falls through to auto-detection.

### 2. New `/use` command

Single new file: `.claude/commands/use.md`.

**`/use <kb-name>`** — sets `.claude/active-kb` to `<kb-name>`.

- `<kb-name>` is the folder basename (matches the QMD collection name). Full paths are not accepted; the basename is the only stable identifier (it is what the QMD collection is keyed on and what the operator sees in `qmd collection list`).
- Validates that `<project-root>/<kb-name>/wiki-config.json` exists.
- On success: write the basename to `.claude/active-kb`, print confirmation including the resolved absolute path.
- On failure: print the list of detected KBs (basenames + paths) and stop. Do not modify the pointer.

**`/use` (no arguments)** — read-only status display.

- Print the currently active KB (basename + absolute path) or "No active KB set" if the pointer is missing.
- List all detected KBs in the project (from the same glob `*/wiki-config.json` + `*/*/wiki-config.json`), one per line, with a marker (e.g. `*`) on the active one.
- Never modifies the pointer.

### 3. Shared KB-resolution preamble

Every command that currently has the "Resolve KB root and QMD availability" preamble (`/ingest`, `/query`, `/wiki`, `/lint`, plus any future command that touches the KB) is updated to use this rule. `/bug` does not currently read KB state and is unchanged.

The new resolution rule, in order:

1. If `.claude/active-kb` exists, read its contents as `<kb-name>`. If `<project-root>/<kb-name>/wiki-config.json` exists, set `<root>` to that directory and read `<collection>` from the JSON. Done.
2. If `.claude/active-kb` exists but is stale (the folder or `wiki-config.json` is gone), print a one-line notice that the pointer is stale and continue to step 3.
3. Glob `*/wiki-config.json` and `*/*/wiki-config.json` from the project root:
   - **Zero hits** — suggest `/setup <path>`. Stop.
   - **Exactly one hit** — use it silently. (No prompt: the single-KB case must remain frictionless.)
   - **Multiple hits** — print a numbered list of detected KBs (basename + absolute path) and stop with a prompt: "Reply with the basename to use for this run, or run `/use <name>` to set it persistently." When the operator replies with a basename, treat it as a one-shot selection: resolve `<root>` and `<collection>` against that KB and continue, but do **not** write `.claude/active-kb`. The pointer is only written by `/use` and `/setup`.
4. QMD availability check (unchanged from current preamble).

This rule is the only place KB resolution lives. CLAUDE.md is updated (§5) to be the canonical, authoritative description; each command's preamble says "follow the KB resolution rule in CLAUDE.md" rather than duplicating the steps. This avoids drift between commands.

### 4. `/setup` and `/ingest` integrations

**`/setup <path>`** — after the existing steps (write `wiki-config.json`, register the QMD collection, index, embed, log), add one final step: write the basename of `<path>` to `.claude/active-kb`. Setting up a KB is an explicit "I'm working on this" signal, so the pointer follows.

**`/ingest <path>`** — after the standard preamble resolves the active KB, identify which KB owns `<path>` by walking up from `<path>` until a `wiki-config.json` is found.

- **`<path>` is inside the active KB** (the common case, e.g. `term-int-foly-inf/raw/foo.pdf` with active = `term-int-foly-inf`) — silent fast path; no prompt. Proceed to ingest as today.
- **`<path>` is inside a different KB** — print: "This source lives in `<other-kb>`, but the active KB is `<active-kb>`." Ask the operator to choose:
  1. Switch active KB to `<other-kb>` and ingest there (writes the pointer).
  2. Ingest into `<other-kb>` just this once without switching the pointer.
  3. Cancel.
  Act on the choice. The previous behavior — silently writing wiki pages into whichever KB happened to be picked — must not occur.
- **`<path>` is not inside any known KB** (no `wiki-config.json` ancestor) — continue with the active KB as before. (Treats free-floating paths as user-meant-active.)

### 5. Documentation updates

**CLAUDE.md — "KB root resolution" section.** Replace the current paragraph with:

- A short description of the active-KB pointer (`.claude/active-kb`, basename, gitignored).
- The 4-step resolution rule from §3 verbatim — this is the canonical, authoritative version. Each command's preamble references this section by name ("follow the KB resolution rule in CLAUDE.md") instead of duplicating the steps, so the rule lives in exactly one place.
- A note that `/use` is the canonical way to switch and `/use` with no args is the canonical way to inspect.

**README.md.**
- Add `/use` to the slash-commands list.
- Add a short "Working with multiple KBs" subsection: `/setup <path>` to onboard a new KB (which also activates it), `/use <name>` to switch later, `/use` to check which is active.

**`.gitignore`.** Add `.claude/active-kb` to the existing `.claude/` block.

## Components and boundaries

- **`.claude/active-kb`** — pointer file. One line, basename only. No frontmatter, no JSON. Single source of truth.
- **`.claude/commands/use.md`** — new command. Two modes (set / read-only status). No side effects beyond writing the pointer in set mode.
- **Shared preamble** — text-level convention, not a runtime module. Each command's markdown is updated to follow the same numbered steps. CLAUDE.md is the canonical reference.
- **`/setup`** — extended with one final pointer-write step.
- **`/ingest`** — extended with cross-KB detection and disambiguation prompt.

Each piece can be reasoned about and changed independently. The pointer file is the only piece of shared mutable state, and its semantics are described in one place (CLAUDE.md).

## Error and edge cases

- **Pointer points to a folder that exists but has no `wiki-config.json`** — treat as stale (same as missing folder). Step 3 takes over.
- **Pointer contains an absolute path or contains slashes** — reject as invalid (commands print: "`.claude/active-kb` should contain a single basename like `term-int-foly-inf`; got `<value>`. Run `/use <name>` to fix."). Stop.
- **Multiple `wiki-config.json` files at different glob depths matching the same KB** — should not happen given the existing layout, but if it does, the deeper match (`*/*/wiki-config.json`) takes precedence over the shallower one (`*/wiki-config.json`) only when their basenames match the active pointer. Otherwise, treat as multi-KB and prompt.
- **`/use felho-prog` while `felho-prog/` is gitignored but missing on disk** — same as "KB does not exist": list the actually-present KBs and stop.
- **`/setup` run on a path whose basename is already the active KB** — write the pointer anyway (idempotent).

## Migration

No automated migration is needed. Operators with one KB will see no behavior change (step 3 with one hit). Operators with multiple KBs will get the disambiguation prompt on the first command after this change lands; running `/use <name>` once resolves it for the session and subsequent sessions.

## Testing

Manual verification in this repo, where multiple KBs already exist:

- `/use` with no args lists `term-int-foly-inf` (and any others present), with the active marker on the right one.
- `/use term-int-foly-inf` writes `.claude/active-kb`; subsequent `/wiki "..."` runs use that KB without prompting.
- `/use does-not-exist` errors and lists valid KBs; pointer unchanged.
- Delete `.claude/active-kb`; `/wiki "..."` prompts (multiple KBs detected).
- Run `/setup felho-prog` and confirm `.claude/active-kb` now reads `felho-prog`.
- Run `/ingest felho-prog/raw/foo.pdf` while active KB is `term-int-foly-inf` and confirm the disambiguation prompt appears.

## Open questions

None at design time. Surface during planning if any arise.
