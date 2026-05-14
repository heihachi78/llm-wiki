# Bug Report Workflow

Use when the user asks to log issues observed during wiki workflow usage.

Only report issues related to:

- ingest workflow
- wiki-only search workflow
- research query workflow
- lint workflow

Ignore unrelated conversation or general repo work.

1. Review the current conversation for:
   - Errors or failures.
   - Incorrect, incomplete, or unexpected behavior.
   - User corrections or complaints.
   - Edge cases handled poorly.
2. If no issues were found, say: `No issues related to wiki workflows were found in this session.`
3. If issues were found, present a numbered list with:
   - Workflow.
   - Problem.
   - Context.
   - Severity: High, Medium, or Low.
4. Ask whether to save the bug report to `bugs.md`.
5. If approved, append to root `bugs.md`:

```markdown
## [YYYY-MM-DD] Bug Report

### Issue N: Short title
- **Command:** workflow-name
- **Severity:** High | Medium | Low
- **Problem:** Description of what went wrong.
- **Context:** What triggered the issue.
- **Status:** Open
```

Append only. Never overwrite previous bug reports.
