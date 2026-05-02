# Wiki source attribution: source-internal locators

**Date:** 2026-05-02
**Status:** Approved (user)
**Affects:** `CLAUDE.md`, `.claude/commands/ingest.md`. (Originally also covered remediation of existing wiki pages — see "Scope reduction" note below.)

## Scope reduction (2026-05-02)

Sections C (remediation of existing wiki pages) and D (re-index + log) are **out of scope** per user direction after the spec was first approved: "you do not need to fix the current knowledge base. just fix the agent prompts." Existing pages in `machine-learning/wiki/` are left as-is. The convention applies prospectively to all future ingests via the updated `ingest.md` and `CLAUDE.md`.

## Problem

Concept and entity pages in this wiki are long-lived: they describe a topic and accrue content from multiple sources over time. The first ingest of the `machine-learning` KB produced 22 concept pages and 2 entity pages that contain **57 source-internal locators on concept pages** and **1 on an entity page** — strings of the form `(slide 11)`, `On slide 2,`, `Slide 11 lists…`, `### Worked example: market segmentation (slide 32)`, etc.

These are unambiguous *today* because each page has exactly one source. As soon as a second source contributes to the same concept page, "slide 11" is ambiguous: slide 11 of which deck? The locators silently rot. With ~22 PDFs queued in `raw/supervised/` and `raw/non-supervised/` for this KB alone, this is a near-term breakage, not a hypothetical.

## Decision

Source-internal locators (slide N, page N, section N, timestamp, line number, etc.) **may appear on source pages only.** Concept, entity, and analysis pages must never use them. Attribution from a concept/entity/analysis page to a source is by **wiki-link to its source page** (e.g. `[01 DM Bevezetés](../sources/01-dm-bevezetes.md)`). The `sources:` frontmatter list plus the source page itself is where slide-level provenance lives — readers who need to verify a claim follow the link.

### Why this rule and not alternatives

- **Per-claim disambiguated locators** (`(01_DM_Bevezetes.pdf, slide 11)`) — rejected for visual noise and edit-time discipline cost on every future integration.
- **Numbered footnotes** — rejected because Obsidian renders footnotes inconsistently inside callouts and lists, and the wiki is read in Obsidian.
- **Status quo (slide N alone)** — the problem under discussion.

## Scope of changes

### A. `CLAUDE.md`

Add a short subsection — placed adjacent to the existing **Source fidelity** section — titled **Cross-page attribution**, containing:

1. The rule above, in canonical wording.
2. One acceptable example.
   - ✅ `[01 DM Bevezetés](../sources/01-dm-bevezetes.md) defines the field as the extraction of patterns…`
3. One unacceptable example.
   - ❌ `Slide 11 defines the field as…`
4. One sentence noting that source pages are exempt: they are 1-1 with a single source, so locators carry no ambiguity there.

### B. `.claude/commands/ingest.md`

- **Step 3 (source page):** unchanged — slide refs are welcome and encouraged.
- **Step 4 (entity pages):** add an explicit "no source-internal locators" line, mirroring the `CLAUDE.md` rule.
- **Step 5 (concept pages):** same addition as Step 4.

### C. Remediation of existing wiki pages

Twenty-four files in `machine-learning/wiki/` need a one-pass cleanup:

- 22 concept pages: every file under `wiki/concepts/`.
- 2 entity pages: `entities/fogarassyne-vathy-agnes.md`, `entities/joseph-rhine.md` (verify with grep at execution time; the grep at design time showed 1 hit but both pages should be inspected).

Mechanical edits per page:

1. Drop `(slide N)` parentheticals.
2. Convert `### X (slide N)` → `### X`.
3. Replace slide-anchored sentence openers (`On slide 2, …`, `Slide 11 lists…`, `Slide 28 ("DM alapfeladatok (2)") describes…`) with one of:
   - source-name attribution: `[01 DM Bevezetés](../sources/01-dm-bevezetes.md) lists…`
   - or simple deletion of the slide opener, letting the prose stand.
   The choice is per-paragraph and at the editor's discretion; the goal is to preserve the claim and its attribution to a source, not its slide number.
4. When in doubt, re-read the corresponding source-page section to preserve fidelity. **Do not add new claims; do not lose existing ones.**

Out of scope for the remediation:

- Restructuring concept-page section headings beyond stripping slide tags. If a heading reads `Worked example: market segmentation`, it stays.
- Footnote infrastructure or any new attribution syntax beyond the existing `[Title](path)` wiki-link pattern.
- Touching source pages at all.
- Touching the `index.md`, `overview.md`, or `log.md`.

### D. Re-index and log

After remediation:

1. `qmd update -c machine-learning && qmd embed`
2. Append a `remediation` entry to `machine-learning/wiki/log.md` describing the pass, the rule, and the file count.

## Verification

The cleanup is complete when:

```bash
grep -rE "slide [0-9]|Slide [0-9]" machine-learning/wiki/concepts machine-learning/wiki/entities
```

returns zero hits. Source pages may still match, and they should — the rule does not apply to them.

A secondary check: every concept and entity page that previously cited a slide must still attribute its claim to a source — either by `[Title](../sources/...)` link in prose or via the `sources:` frontmatter list. No claim should become orphaned.

## Non-goals

- Codifying attribution conventions for other source-internal locators that don't exist yet (chapter, paragraph). The rule already covers them by category ("source-internal locators"); no enumeration needed.
- Retroactively applying this convention to other KBs (none exist alongside `machine-learning` yet).
- Tooling to automatically detect violations during ingest. The grep above is sufficient as a manual check; lint may add it later if drift recurs.

## Acceptance

User approved the design in the brainstorming session on 2026-05-02. Implementation proceeds via `superpowers:writing-plans`.
