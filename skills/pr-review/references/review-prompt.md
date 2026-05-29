# Review Agent Dispatch Prompt

Template for dispatching specialist review agents. The main model fills in the bracketed sections.

---

## Review: [agent role]

### PR Context
- **PR:** #[number] — [title]
- **Repository:** [owner/repo]
- **Branch:** [head] → [base]
- **Description:** [PR summary — what was built and why]

### Changed Files
[List of files with change type: added/modified/deleted]

### Platform Context

{{platform_context}}

If no platform context is provided, skip this section.

### Diff
[The relevant portion of the diff for this agent's focus area. For focused agents like silent-failure-hunter, include only files with error handling. For broad agents like code-quality-reviewer, include the full diff.]

### Instructions
Review ONLY what the PR changed. Do not flag:
- Pre-existing issues on unchanged lines
- Issues a linter, typechecker, or CI would catch
- Style or formatting preferences
- General observations that aren't actionable

For each finding, include:
- File path and line number
- Code snippet showing the issue
- Evidence explaining why this is a real issue
- Specific suggestion for fixing it

If no issues found, say so clearly.

---

## Confidence Scoring Prompt (Haiku)

Used to score each finding after specialist agents return.

### Score This Finding

**Finding from [agent name]:**
[finding text including file, line, evidence, suggestion]

**PR diff context:**
[relevant code snippet from the diff]

Score this finding 0-100 based on:
- Is the evidence specific (file, line, code snippet)? (+20)
- Is the issue in code the PR actually changed? (+20)
- Would a senior engineer flag this in review? (+20)
- Is this a real bug or just a preference? (+20 for real bug)
- Could CI catch this instead? (-20 if yes)

Return ONLY: `score: [0-100]` and one sentence explaining why.
