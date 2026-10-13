---
name: implementer-opus
description: "Builds one ticket again on Opus after the implementer failed on it, in its own worktree, from the first attempt's report and worktree. Never commits; the session commits. Reports its status, its worktree, a suggested commit subject and the evidence. The filled prompt is the complete task."
model: opus
effort: high
isolation: worktree
tools: Read, Glob, Grep, Edit, Write, Bash
---
