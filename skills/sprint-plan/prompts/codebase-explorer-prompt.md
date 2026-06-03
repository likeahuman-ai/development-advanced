# Codebase Explorer Prompt Template

Use this template when dispatching `codebase-explorer` agents. Launch 2-3 in parallel, each with a different mode.

## Mode 1: Architecture Mapping

> Map the architecture of the project: module structure, startup/initialisation flow, key abstractions, how components communicate. Report file paths for everything you read.

## Mode 2: Pattern Matching

> Find features similar to [FEATURE] in the project. Document coding conventions, file organization patterns, testing patterns, and UI/interface patterns. Report file paths.

## Mode 3: Integration Analysis

> Analyze where [FEATURE] would integrate into the project. What existing modules does it touch? What are the constraints? What dependencies exist? What could break? Report file paths.

## Usage

Replace `[FEATURE]` with the specific feature being planned or built. The main model reads key files the agents identify — don't rely solely on agent summaries.

`/sprint-plan` consumes these findings to:

- **Write the Sprint Plan Architecture shape-of-change pointer** — a 2-3 line "shape of the change" against existing structure (not a full architecture block). Mode 1 (Architecture Mapping) and Mode 3 (Integration Analysis) supply the baseline this delta points at.
- **Capture ADRs** — Mode 3 surfaces the constraints and break points that motivate decisions. Express each ADR's Scope as a **coarse subsystem boundary** (e.g. "auth", "ingest pipeline"), never as file paths or globs.
- **Detect story conflicts** — when a new want collides with an existing one, the explorer findings ground the WHY that gets recorded in an ADR.

These findings are exploration inputs: the Sprint Plan, ADRs, and Stories **reference** the relevant modules and decisions — they do not reproduce the agents' reports verbatim. Pull the few load-bearing file pointers into the artifacts; leave the rest in the agent summaries.
