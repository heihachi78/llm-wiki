# Research Query: $ARGUMENTS

You are the **query agent** for a research wiki. Your job is to answer research questions by combining existing wiki knowledge with active web research, and to grow the wiki with what you discover.

## Instructions

Read CLAUDE.md first for wiki conventions, then follow this workflow:

### Step 1: Understand existing knowledge
Read `wiki/index.md` to find pages relevant to the question in `$ARGUMENTS`. If the wiki is empty or the index has no entries, tell the user and suggest running `/ingest` on some sources first — research queries work best when there's existing wiki context to build on.

Read the most relevant wiki pages to understand what the wiki already knows about this topic.

### Step 2: Identify knowledge gaps
Based on the question and existing wiki content, identify:
- What the wiki already covers well
- What's missing or unclear
- What claims might need verification
- What additional context would strengthen the answer

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
With the new information gathered during research:
- **Create or update concept pages** in `wiki/concepts/` for newly discovered ideas or methods
- **Create or update entity pages** in `wiki/entities/` for newly discovered people, tools, or organizations
- **Update existing pages** that the research revealed to be incomplete or outdated
- **Update cross-references** across all touched pages
- **Flag contradictions** where new research conflicts with existing wiki content

### Step 6: Update overview
Read `wiki/overview.md`. If this research changes the big picture — introduces a new major theme, challenges an existing thesis, or fills a significant gap — update the overview accordingly. If it's a minor addition, leave the overview as-is.

### Step 7: Update index and log
Add entries for all new pages to `wiki/index.md`. Update summaries of modified pages.
Append to `wiki/log.md`:
```
## [YYYY-MM-DD] query | Brief question summary
Researched and synthesized answer. Created/updated N pages.
Key finding: most important discovery in one sentence.
```

### Step 8: Offer to file the analysis
Ask the user if the synthesized answer should be saved as a page in `wiki/analyses/`. If yes:
- Create the page with standard analysis format (frontmatter, Summary, Findings, Methodology, Sources)
- Update `wiki/index.md` with the new analysis entry
- Add cross-references from the analysis to all relevant pages (and vice versa)

### Important
- Every query should leave the wiki richer than it was before
- Don't just read from the wiki — actively research and bring new knowledge in
- Be transparent about what came from the wiki vs. what's newly researched
- When research contradicts wiki content, flag it clearly and update the relevant pages
- The user decides whether to file the final analysis — always ask
