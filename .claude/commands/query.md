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

### Step 3: Active research
Spawn research subagents (using the Agent tool) to investigate knowledge gaps. Each subagent should have a focused research task:
- **Web search agent:** Search the web for current information, recent developments, key papers or articles on the topic. Use WebSearch and WebFetch tools.
- **Cross-check agent:** If the wiki makes specific claims, verify them against external sources. Look for contradictions or updates.
- **Deep-dive agent (optional):** For complex questions, spawn an agent to research a specific subtopic in depth.

Launch independent research agents in parallel for efficiency.

### Step 4: Synthesize answer
Combine wiki knowledge and new research into a comprehensive answer:
- Lead with a clear, direct answer to the question
- Support with evidence from both wiki pages and new research
- Cite sources: link to wiki pages and note external URLs
- Flag any contradictions between wiki content and new findings
- Note confidence levels — what's well-established vs. tentative

Present the answer to the user.

### Step 5: Update the wiki
With the new information gathered during research, apply the source-fidelity rules in CLAUDE.md. Web-research results *are* allowed to introduce new claims — that's the point of `/query` — but each new claim must be attributable to the URL it came from, and existing source-derived content must not be silently rewritten.

- **Create or update concept pages** in `<root>/wiki/concepts/` for newly discovered ideas or methods. Tie each new claim to the web source it came from (e.g. `According to <url>, ...`) so wiki-internal vs. web-derived content stays distinguishable.
- **Create or update entity pages** in `<root>/wiki/entities/` for newly discovered people, tools, or organizations — only those actually named in the research output, not extrapolations.
- **Update existing pages** that the research revealed to be incomplete or outdated. When extending a page, *add* new web-attributed material; do not paraphrase or "tidy up" prior source-derived content.
- **Update cross-references** across all touched pages.
- **Flag contradictions** where new research conflicts with existing wiki content. Do not resolve contradictions by silently editing the older content — surface them.
- **No invented quotes.** If you write a `> "..."` block, it must be a verbatim copy from the wiki source page or the web source you actually fetched. Do not reconstruct quotes from memory.

### QMD re-index (after Step 5, before Step 6)
If QMD available, re-index immediately after all page writes so the index reflects updated semantics:
```bash
qmd update -c <collection>
qmd embed
```
Note: `qmd embed` has no `-c` flag — it embeds all collections. This step is critical in `/query` because updated pages may have significantly changed semantics.

### Step 6: Update overview
Read `<root>/wiki/overview.md`. If this research changes the big picture — introduces a new major theme, challenges an existing thesis, or fills a significant gap — update the overview accordingly. If it's a minor addition, leave the overview as-is.

### Step 7: Update index and log
Add entries for all new pages to `<root>/wiki/index.md`. Update summaries of modified pages.
Append to `<root>/wiki/log.md`:
```
## [YYYY-MM-DD] query | Brief question summary
Researched and synthesized answer. Created/updated N pages.
Key finding: most important discovery in one sentence.
```

### Step 8: Offer to file the analysis
Ask the user if the synthesized answer should be saved as a page in `<root>/wiki/analyses/`. If yes:
- Create the page with standard analysis format (frontmatter, Summary, Findings, Methodology, Sources)
- Update `<root>/wiki/index.md` with the new analysis entry
- Add cross-references from the analysis to all relevant pages (and vice versa)

### Important
- Every query should leave the wiki richer than it was before
- Don't just read from the wiki — actively research and bring new knowledge in
- Be transparent about what came from the wiki vs. what's newly researched
- When research contradicts wiki content, flag it clearly and update the relevant pages
- The user decides whether to file the final analysis — always ask
- Never nest markdown tables inside list items — Obsidian cannot render indented tables. Use flat bullet lists instead.
- You can always go back to the raw source files and read them, because its not guaranteed, that all the important information from the original source file is present in the wiki.

