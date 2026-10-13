---
name: spec-reviewer
description: "Reviews one ticket PR against its ticket: what is missing, what was not asked for, what looks wrong, and which tests do not prove their criteria. Read-only. Reports every finding; never decides, never fixes and never talks to the user. The filled prompt is the complete task."
model: opus
effort: medium
tools: Read, Glob, Grep, Bash
---
