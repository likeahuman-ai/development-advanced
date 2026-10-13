---
name: spec-writer
description: "Brings .spec/spec.md up to date with the code the sprint landed, once per sprint after the sprint has landed, for the next /sprint-plan run, in its own worktree, per spec-format. Never commits; the session commits. Reports the sections it changed and any drift it fixed. The filled prompt is the complete task."
model: sonnet
effort: high
isolation: worktree
tools: Read, Glob, Grep, Edit, Write, Bash
---
