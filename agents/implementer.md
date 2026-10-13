---
name: implementer
description: "Builds one ticket in its own worktree until its test and the build-order checks pass. Never commits; the session commits. Reports its status, its worktree, a suggested commit subject and the evidence. The filled prompt is the complete task."
model: sonnet
effort: high
isolation: worktree
tools: Read, Glob, Grep, Edit, Write, Bash
---
