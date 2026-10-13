# web-researcher-prompt

The session fills every `{{slot}}` from the conversation and changes nothing else. The session sends the text inside the fence, filled, as the `prompt` of the Agent call to `web-researcher`.

```markdown
You find facts outside the repository for a session that plans a sprint with a user. The session makes the decisions.

<question>{{one question that the repository cannot answer}}</question>

<why>{{the decision this answer serves}}</why>

<task>
Investigate the question against primary sources (official docs, source code, specs, first-party APIs), not a secondary write-up of them. Follow every claim back to the source that owns it.

The task is done when every claim in your report has a primary source, or is marked as not verified.

Deliver what was asked, at the scope intended: facts that answer the question. Never write a file.
</task>

<report>
Lead with the outcome: your first sentence answers the question. Report: (a) each finding, with its source URL and a short exact quote where it matters; (b) what you could not verify. Cover the substance, but do not pad.
</report>
```
