---
name: tickets
description: "Turn an approved PRD into AI-ready GitHub Issues with implementation detail. Use when participant has an approved PRD, says 'break this down', 'create issues', 'make tickets', 'turn this into tasks', or has a completed PRD that needs implementation planning."
argument-hint: "Path to PRD (optional — auto-detects most recent)"
---

# /tickets — PRD to GitHub Issues

You are turning an approved PRD into actionable, AI-ready GitHub Issues. You work through six phases: prerequisites, codebase re-exploration, architecture & decomposition, issue creation, build ordering, and PRD status update. You are mostly autonomous — one approval gate before creating issues.

**Initial request:** $ARGUMENTS

---

## Phase 0: Prerequisites

**Goal:** Find and confirm the PRD, ensure it's committed.

1. **Find the PRD:**
   - If `$ARGUMENTS` contains a path, use that.
   - Otherwise, check `.prd/` for the most recently modified `.md` file.
   - Confirm with the user: "I found `.prd/prd-v1.md` — is this the one?"

2. **Check git status:**
   - Is the PRD committed? Run `git status` to check.
   - If uncommitted: commit it. PRDs need version history before tickets reference them.
     ```
     git add .prd/prd-v{N}.md
     git commit -m "docs: add PRD for {feature name}"
     ```

3. **Check GitHub remote:**
   - Run `git remote get-url origin` to check if a GitHub remote exists.
   - If no remote:
     - Ask the participant for a repo name (suggest the project folder name).
     - Create the repo: `gh repo create {name} --private --source=. --push`
     - Tell the participant: "I've created a GitHub repository for your project."
   - If remote exists, continue.

4. **Read and parse the PRD.** Identify:
   - Core features/epics to decompose
   - Architecture decisions already made
   - Scope boundaries (what's in, what's out)
   - Success metrics (tickets must cover these)

---

## Phase 1: Codebase Re-exploration

**Goal:** Get fresh codebase context. The PRD may have been written days ago — the code may have changed.

**No gate — this phase is autonomous.**

1. Launch 2-3 `codebase-explorer` agents (sonnet, parallel). Use the prompt template from `skills/tickets/references/explorer-prompt.md` — focus agents on areas the PRD touches.

2. Read key files the agents identified.
3. Compare exploration findings against the PRD's architecture section:
   - **Consistent** → proceed silently.
   - **Contradiction** → flag to user. The PRD has priority unless the code reveals an anti-pattern the PRD didn't account for.

---

## Phase 2: Architecture & Decomposition

**Goal:** Design the implementation and break it into right-sized tickets.

1. Launch `code-architect` agents (inherit, parallel). Each agent takes a different epic/feature from the PRD. Use the prompt template from `skills/tickets/references/architect-prompt.md`.

2. Read the agents' findings. Assemble the full breakdown.

3. **Decide structure based on size:**
   - **< 8 tickets** → flat structure. All issues at the same level, labels differentiate.
   - **8+ tickets** → hierarchical. Epic issues as parents, feature issues group tasks, task issues are atomic.

4. **Present the breakdown** to the user in a grouped format:

   ```
   ## Breakdown: [Feature Name] ([N] tickets)

   ### Epic: [Name] — [priority]

   **Feature: [Name]** — [priority]
   - [Ticket title] [complexity] — [priority]
   - [Ticket title] [complexity] — [priority]

   **Feature: [Name]** — [priority]
   - [Ticket title] [complexity] — [priority]
   ```

   Include key details: what each ticket covers, dependencies, and anything the user should know.

**Gate:**
The user must approve the breakdown before you create issues. Ask: "Ready to create these as GitHub Issues? Any changes first?"

---

## Phase 3: Create GitHub Issues

**Goal:** Create well-structured GitHub Issues with AI-ready content.

### Determine the repository

Check `git remote -v` to get the GitHub repository. Use the `gh` CLI for all GitHub operations.

### Ensure labels exist

Before creating issues, ensure the required labels exist. Check with `gh label list` and create any missing ones:

**Hierarchy:** `epic:{name}`, `feature:{name}`
**Priority:** `blocker`, `important`, `nice-to-have`, `low`
**Version:** `v1`, `v2`, `v3`, etc.
**Type:** `bug`, `refactor`, `docs`
**Complexity:** `S`, `M`, `L`
**Workflow:** `build-order`

### Create issues

For each ticket in the approved breakdown:

**Issue body format:** Use the template from `skills/tickets/references/ticket-template.md`.

**Issue creation order:**
1. Create milestone if the PRD warrants one.
2. Create epic issues first (if hierarchical).
3. Create feature issues, referencing epic.
4. Create task issues, referencing feature and adding dependency links.
5. Apply labels: priority + version + complexity (+ hierarchy labels if applicable).

Use `gh issue create` with `--title`, `--body`, and `--label` flags. Use HEREDOCs for the body.

### Present summary

After all issues are created, present a grouped summary with URLs:

```
## Created: [Feature Name] ([N] tickets)

### Epic: [Name] (#[number])

**Feature: [Name] (#[number])** — [priority]
- #[number] [Title] [complexity] — [priority]
- #[number] [Title] [complexity] — [priority]

**Feature: [Name] (#[number])** — [priority]
- #[number] [Title] [complexity] — [priority]
```

---

## Phase 4: Build Order

**Goal:** Produce a build-order issue so /build knows the implementation sequence.

Using the code-architect findings (file paths, creates/consumes, dependencies, complexity) and the codebase context from Phase 1:

1. **Build the dependency graph.** For each ticket, map what it creates and what it consumes. Use the code-architects' Creates/Consumes fields.

2. **Sequence tickets.** Apply these rules in order:
   - HARD dependencies are inviolable — producer before consumer.
   - Foundational work (types, schemas, shared utilities) before features that use them.
   - Blockers before important before nice-to-have at the same dependency level.
   - Tickets touching the same files should be adjacent (reduces context switching).
   - SOFT dependencies prefer producer-first but can be reordered if it improves grouping.

3. **Group into PRs.** Group by coupling, not line count:
   - Tickets sharing a runtime boundary belong in the same PR.
   - Each PR must be independently reviewable and testable.
   - Note estimated line count per PR for reference, but do not use it as the grouping criterion.

4. **Create the build-order issue.**
   - Title: `Build Order: [feature/version]`
   - Labels: `build-order`, version label
   - Body format:

   ```
   ## Dependency Graph

   #[number] creates:
     - [file/pattern] ([description])

   #[number] consumes:
     - [file/pattern] (from #[source]) [HARD]
     - [pattern] (from #[source]) [SOFT]

   ## Build Sequence

   1. #[number] — [title] ([complexity], [priority]) — [one-line reasoning]
   2. #[number] — [title] ([complexity], [priority]) — [one-line reasoning]

   ## PR Grouping

   PR 1: #[number] + #[number]
     Coupling: [shared runtime boundary rationale]
     Independently reviewable: yes — [reason]

   ## Flags

   - [reorderable pairs, independent tickets, or other sequencing notes]
   ```

   - Pin the issue: `gh issue pin [number]`

No additional user gate — the user already approved the breakdown in Phase 2. The build order is a deterministic consequence of that breakdown.

**Dependency strength:**
- **HARD** — ticket B cannot compile/run without ticket A's output. Must be sequential.
- **SOFT** — ticket B works without ticket A's output, but is better/cleaner with it. Can be reordered if needed.

---

## PRD Status Update

After all issues are created and the build-order issue is pinned, update the PRD frontmatter from `status: draft` to `status: built`. Read the PRD file, replace `status: draft` with `status: built` in the YAML frontmatter, and write it back.

---

## Key Principles

- **One ticket = one independently verifiable change** = roughly one PR.
- **AI-ready content** — explicit file paths, verifiable criteria, verification commands. No business justification (that's in the PRD).
- **No CLAUDE.md duplication** — tickets contain only the delta specific to this task.
- **Acceptance criteria are testable**, not subjective.
- **Dependencies are explicit** — blocked-by and blocks references using issue numbers.
- **Complexity is AI resource cost** (S/M/L), never time estimates.
