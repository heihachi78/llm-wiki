# Research Query: $ARGUMENTS

You are the **query agent** for a research wiki. Your job is to answer research questions by combining existing wiki knowledge with active web research, and to grow the wiki with what you discover.

## Instructions

Read CLAUDE.md first for wiki conventions, then follow this workflow step-by-step:

### Preamble: Resolve KB root and QMD availability

Follow the **KB resolution rule** in `CLAUDE.md` (section "KB root resolution"). It defines steps 1-4 (read `.claude/active-kb`; handle stale pointer; auto-detect via glob with zero / one / many cases; check QMD availability). The result of those steps is `<root>` (KB folder), `<collection>` (QMD collection name), and a boolean for QMD availability.

Use `<root>/` everywhere in this command. If QMD is not available, skip all QMD steps and note `[QMD not available — semantic steps skipped]` once in your response.

### Step 1: Understand existing knowledge
Read `<root>/wiki/index.md` to find pages relevant to the question in `$ARGUMENTS`. If the wiki is empty or the index has no entries, tell the user and suggest running `/ingest` on some sources first — research queries work best when there's existing wiki context to build on.

Read the most relevant wiki pages to understand what the wiki already knows about this topic.

### Step 2: Identify knowledge gaps
Based on the question and existing wiki content, identify:
- What the wiki already covers well
- What's missing or unclear
- What claims might need verification
- What additional context would strengthen the answer

**If QMD available:** additionally run:
```bash
qmd query "<question from $ARGUMENTS>" -c <collection> -n 10 --json
```
A page returned by QMD with high relevance is treated as existing knowledge — this reduces false gap identification for topics covered in the wiki under different terminology.

### Step 3: Re-read relevant raw sources
The wiki is a *summary* of the raw sources, and summaries miss things. Before reaching for the web, go back to the original material — `/ingest` may not have captured every detail that turns out to be relevant to *this specific* question, and parts of the source may have been processed without sufficient care or detail for the angle the user is now asking about.

- For each wiki page identified as relevant in Step 1 (and any high-relevance hit from the QMD query in Step 2), look at its `sources:` frontmatter. Each entry is a filename — the corresponding raw file lives in `<root>/raw/` (typically with the same basename, possibly different extension).
- Re-read those raw files directly with the question in mind. Scan for claims, examples, definitions, quotes, numbers, or caveats that are relevant to `$ARGUMENTS` but are missing from, underemphasised in, or paraphrased away on the existing wiki pages.
- For longer raw sources (slide decks, papers, transcripts), do not skim only the sections the wiki already cites — check the parts that the wiki currently says little about, since that is exactly where ingest gaps tend to hide.
- Source fidelity rules from CLAUDE.md apply: verbatim quotes only, no extrapolation, no filling in from background knowledge. If the raw does not say it, do not write it.
- Note findings for use in Step 6 (synthesis) and Step 7 (wiki updates). Distinguish raw-derived findings from web-derived ones — they attribute differently.

This step is the primary way `/query` strengthens the wiki for source-grounded questions; web research in Step 4 is supplementary.

### Step 4: Active research
Spawn research subagents (using the Agent tool) to investigate knowledge gaps that the raw sources did not resolve. Each subagent should have a focused research task:
- **Web search agent:** Search the web for current information, recent developments, key papers or articles on the topic. Use WebSearch and WebFetch tools.
- **Cross-check agent:** If the wiki or raw sources make specific claims, verify them against external sources. Look for contradictions or updates.
- **Deep-dive agent (optional):** For complex questions, spawn an agent to research a specific subtopic in depth.

Launch independent research agents in parallel for efficiency. Skip this step entirely if Step 3 already produced a sufficient answer and the question does not call for external context.

### Step 5: Synthesize answer
Combine wiki knowledge and new research into a comprehensive answer:
- Lead with a clear, direct answer to the question
- Support with evidence from wiki pages, raw sources re-read in Step 3, and new web research from Step 4
- Cite sources: link to wiki pages, name the raw source for raw-derived points, note external URLs for web-derived points
- Flag any contradictions between wiki content, raw sources, and new findings
- Note confidence levels — what's well-established vs. tentative

Present the answer to the user.

### Step 6: Update the wiki
Apply the source-fidelity rules in CLAUDE.md. Two kinds of new material may have surfaced; they attribute differently:

- **Raw-derived additions** (from Step 3): material that was already in `<root>/raw/` but missing from the wiki. These follow the normal ingest rules — attribute to the source page via wiki-link in `sources:` frontmatter and inline links. Verbatim quotes only. On source pages, slide/page locators are fine; on concept/entity/analysis pages, never use source-internal locators (CLAUDE.md "Cross-page attribution" rule).
- **Web-derived additions** (from Step 4): tie each new claim to the URL it came from (e.g. `According to <url>, ...`) so wiki-internal vs. web-derived content stays distinguishable.

Then:
- **Update source pages** in `<root>/wiki/sources/` with anything newly extracted from raw in Step 3 — this is where the wiki's grasp of the source actually deepens. Append to existing sections rather than rewriting; bump `date_updated`.
- **Create or update concept pages** in `<root>/wiki/concepts/` for newly surfaced ideas or methods (from raw or web). Only create pages for things actually named in the material, not extrapolations.
- **Create or update entity pages** in `<root>/wiki/entities/` for newly surfaced people, tools, or organizations — same rule.
- **Update existing pages** that Step 3 or Step 4 revealed to be incomplete. *Add* new material; do not paraphrase or "tidy up" prior source-derived content under cover of the new pass.
- **Update cross-references** across all touched pages (bidirectional).
- **Flag contradictions** where new findings conflict with existing wiki content. Do not resolve contradictions by silently editing the older content — surface them.
- **No invented quotes.** If you write a `> "..."` block, it must be a verbatim copy from the raw source you re-read, the wiki source page, or the web source you actually fetched. Do not reconstruct quotes from memory.

### QMD re-index (after Step 6, before Step 7)
If QMD available, re-index immediately after all page writes so the index reflects updated semantics:
```bash
qmd update -c <collection>
qmd embed
```
Note: `qmd embed` has no `-c` flag — it embeds all collections. This step is critical in `/query` because updated pages may have significantly changed semantics.

### Step 7: Update overview
Read `<root>/wiki/overview.md`. If this research changes the big picture — introduces a new major theme, challenges an existing thesis, or fills a significant gap — update the overview accordingly. If it's a minor addition, leave the overview as-is.

### Step 8: Update index and log
Add entries for all new pages to `<root>/wiki/index.md`. Update summaries of modified pages.
Append to `<root>/wiki/log.md`:
```
## [YYYY-MM-DD] query | Brief question summary
Researched and synthesized answer. Re-read N raw sources; created/updated M wiki pages.
Key finding: most important discovery in one sentence.
```

### Step 9: Offer to file the analysis
Ask the user if the synthesized answer should be saved as a page in `<root>/wiki/analyses/`. If yes:
- Create the page with standard analysis format (frontmatter, Summary, Findings, Methodology, Sources)
- Update `<root>/wiki/index.md` with the new analysis entry
- Add cross-references from the analysis to all relevant pages (and vice versa)

### Important
- Every query should leave the wiki richer than it was before
- Always re-read the relevant raw sources (Step 3) — ingest summaries are lossy, and the wiki may simply be missing the part that answers *this* question. Treat raw re-reads as a first-class source of new knowledge, not a fallback.
- Be transparent about what came from the wiki, what came from re-reading raw, and what came from web research — the three are attributed differently
- When findings contradict wiki content, flag it clearly and update the relevant pages — never silently rewrite prior source-derived claims
- The user decides whether to file the final analysis — always ask
- Never nest markdown tables inside list items — Obsidian cannot render indented tables. Use flat bullet lists instead.

