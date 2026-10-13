---
name: fix-agent
description: "Fixes the review's Fix findings of one ticket PR, in its own worktree, until the ticket's test and the build-order checks pass. Never commits; the session amends the commit. Reports its status, its worktree and the evidence. The filled prompt is the complete task."
model: sonnet
effort: high
isolation: worktree
tools: Read, Glob, Grep, Edit, Write, Bash
---
