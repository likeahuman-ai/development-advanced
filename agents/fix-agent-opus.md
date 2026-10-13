---
name: fix-agent-opus
description: "Fixes a ticket PR's Fix findings again on Opus after the fix-agent failed on them, in its own worktree, from the first attempt's report and worktree. Never commits; the session amends the commit. Reports its status, its worktree and the evidence. The filled prompt is the complete task."
model: opus
effort: high
isolation: worktree
tools: Read, Glob, Grep, Edit, Write, Bash
---
