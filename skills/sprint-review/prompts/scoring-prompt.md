# Confidence Scoring Prompt

Used to score each finding after the specialist review agents return. Dispatched as a single scoring agent (see `/sprint-review` Phase 4).

## Score This Finding

**Finding from [agent name]:**
[finding text including file, line, evidence, suggestion]

**PR diff context:**
[relevant code snippet from the diff]

Score this finding 0-100 as a **priority** — how strongly we should act on it — by blending **confidence** (is it real?) with **impact** (how much does it hurt?):

**Confidence (is it real?):**
- Evidence is specific (file, line, code snippet)
- The issue is in code the PR actually changed
- A senior engineer would flag it — a real bug, not a preference
- A linter / typechecker / CI would NOT already catch it

**Impact (how much it hurts):**
- **User-facing harm** — crash, data loss, wrong result, broken flow, exposed secret/PII — scores HIGH even at moderate confidence (missing it is expensive)
- **Internal-only** — code hygiene, style, an internal log line or URL — scores LOWER unless confidence is high (missing it is cheap; false alarms cost attention)

A high-impact user-facing bug you are moderately sure about outranks a trivial internal nit you are certain about. Impact is folded INTO the one score — there is no separate user-facing/internal threshold.

Score bands (these drive what happens to the finding):
- **90-100** — certain and impactful → must fix before merge
- **75-89** — verified real → fix this cycle (published as a PR comment)
- **50-74** — real but not worth blocking → findings backlog (`.sprint/backlog.md`)
- **<50** — noise / preference / low-confidence-and-low-impact → dropped

Return ONLY: `score: [0-100]` and one sentence explaining why.
