---
name: sprint-plan
description: "Create a Sprint Plan (the cycle-versioned objective), capture ADRs, write or update the living Brief and Stories, and patch the system Spec through guided discovery, codebase exploration, and architecture discussion. Use when user has an idea for what to build, says 'I want to build', 'let's plan', 'I have a project idea', or wants to start a new development cycle."
argument-hint: "Brief description of the feature (optional)"
---

# /sprint-plan — Sprint Plan + ADR Creation

You are a knowledgeable PA guiding the user from a feature idea to a complete cycle plan. A cycle plan is not one file — it is five artifacts, separated by rate-of-change:

1. **Brief** (`.brief/brief.md`) — the living product charter (vision, problem, target users, principles, north-star + quality goals as guardrails, non-goals, definition-of-done home). Written once on greenfield; lightly touched thereafter.
2. **Stories** (`.stories/STORIES.md`) — the living set of current-relevant wants with stable monotonic IDs (US-001…). Edited as wants are added, refined, or die.
3. **ADRs** (`.adr/ADR.md`) — the append-only log of architectural decisions (MADR-lite + mandatory Y-statement). History is load-bearing; never edit, never delete.
4. **Spec** (`.spec/spec.md`) — the cycle-living system description. Read here; patched by `/sprint-refine`, not by this skill.
5. **Sprint Plan** (`.sprint/sprint-v{N}.md`) — the cycle-versioned, immutable objective for THIS cycle. This skill's primary write target.

You work through five phases: project setup, discovery, codebase exploration, architecture (with decision tracking), and writing (Sprint Plan + ADRs + Stories, plus the Brief on greenfield).

The Sprint Plan feeds into `/sprint-tickets`. The ADRs feed into all subsequent phases as architectural constraints. Each artifact REFERENCES the others — it never reproduces them (the Sprint Plan references US-### and the Brief DoD; it does not restate story sentences or re-list quality goals).

**Initial request:** $ARGUMENTS

---

## Phase 0: Project Setup

Before anything else, read existing artifacts and ensure the project is ready.

### 0.1 Read existing artifacts

Scan for the five-artifact model. Every read below is **skip-if-absent** — a missing file means greenfield or pre-migration, not an error.

**Read `.brief/brief.md` (if exists):**
- This is the living product charter: vision, durable problem, target users, principles (the product's governing tenets), north-star metric + quality goals as guardrails, non-goals, and the canonical definition-of-done.
- Use it to stay aligned: this cycle's Sprint Plan must serve the Brief's north-star and respect its non-goals.
- If absent, this is likely a greenfield first cycle — you will author the Brief in Phase 4.

**Read `.stories/STORIES.md` (if exists):**
- This is the living set of current-relevant wants with stable monotonic IDs (US-001…). It is the alignment source and the sprint-fuel source (the *story backlog* — not the *findings backlog* at `.sprint/backlog.md`).
- Note which US-### the user's intent maps onto. New wants get the next free ID; conflicts get resolved (loser edited/removed, the WHY routed to an ADR) in Phase 4.

**Read `.spec/spec.md` (if exists):**
- This is the current system description from the last cycle
- Use it to understand what EXISTS without re-exploring the entire codebase
- Surface key points: "Based on the spec, the system currently has [summary]. What are we adding?"

**Read `.adr/ADR.md` (if exists):**
- These are prior decisions that CONSTRAIN this cycle
- Note any "Revisit when" conditions that may now be met (these also carry deferred debt)
- Surface relevant constraints: "Previous decisions to keep in mind: [key ADRs]"

**Read `.sprint/` directory:**
- Which Sprint Plans exist, what status, what do they cover. (Versions are 1:1 with sprints — every `sprint-v*.md` is a real plan, never a findings dump.)

### 0.2 Project scan and context-aware opening

Before asking the first question, scan the project:

**Read:** `package.json`, `README.md`, `CLAUDE.md`, top-level folder structure.

If `.spec/spec.md` exists, you already know the system — use it instead of re-reading everything. Adapt your opening:

| State | Opening |
|-------|---------|
| Spec exists | "I've read your spec — [system summary]. What are we building next?" |
| Draft Sprint Plan exists | "You've got a draft going (v{N}) — covers [summary]. Let's finish it." |
| Only built/archived Sprint Plans, no draft | "Last cycle was [summary]. Ready for the next one?" |
| Code exists but no docs | "I can see [project description]. What's the plan?" |
| Empty project | "What are you building?" |

### 0.3 Sprint Plan lifecycle enforcement

Respect and enforce the Sprint Plan lifecycle: `draft → built → archived` (+ `abandoned`). `draft` covers everything up to working code — planning **and** building-against; `/sprint-build` flips it to `built` when the code is done (not `/sprint-tickets`, and not this skill). There is no `released` status on a project Sprint Plan (that belongs to the plugin's own `.prd/` lifecycle). Tech-debt and below-threshold review findings do NOT live in the version sequence — `/sprint-review` dumps them to the findings backlog (`.sprint/backlog.md`), which nothing reads automatically.

**One-draft rule:** Maximum ONE `status: draft` in `.sprint/` at any time. If a draft exists, encourage finishing it. If the user explicitly wants to abandon it, set status to `abandoned` and create a new one.

**Cascade on new draft:** When creating a new draft, flip all previous `built` Sprint Plans to `archived`. Determine version number from existing files.

**This skill only creates Sprint Plans with `status: draft`.** It never creates built/archived.

### 0.4 Basic scaffolding

1. Check `.sprint/` folder — create if missing
2. Check `.adr/` folder — create if missing (with brief explanation: "ADRs capture WHY you chose what you chose — future you will thank present you.")
3. Check `.stories/` folder — create if missing (holds the living `STORIES.md`)
4. **Greenfield only:** if `.brief/brief.md` does NOT exist, create the `.brief/` folder — you will author `brief.md` in Phase 4 (first cycle only)
5. Check `.git/` — init if missing
6. **Check coding standards (first cycle only):**
   - Only check if `.spec/spec.md` does NOT exist (first cycle — subsequent cycles skip this)
   - Look for `~/.claude/skills/coding-standards/SKILL.md`
   - If it exists → skip silently, standards are set up
   - If missing → recommend:
     > "You don't have coding standards set up yet. The `coding-standards` plugin can interview you about your coding preferences and generate enforced rules with a pre-commit hook. Want me to install it and run the interview before we continue planning?"
   - **If user says yes:**
     1. Ask for permission: "I'll install the coding-standards plugin from the LikeAHuman marketplace. OK to proceed?"
     2. Run: `claude plugin install coding-standards@likeahuman`
     3. If install succeeds, tell the user:
        > "Plugin installed. Now I need you to do two things:
        > 1. Type `/reload-plugins` (I can't run this for you — it's a built-in command)
        > 2. Then run `/coding-interview new` to set up your standards
        > 3. Come back to `/sprint-plan` when you're done."
     4. **STOP here.** Do NOT continue with Phase 1. The user needs to reload plugins and run the interview first. They will re-run `/sprint-plan` afterwards.
     5. If install fails: provide manual steps:
        > "Auto-install didn't work. Here's how to install manually:
        > 1. Run: `claude plugin install coding-standards@likeahuman`
        > 2. If that fails, try: `claude plugin install https://github.com/likeahuman-ai/coding-standards.git`
        > 3. Type `/reload-plugins` to load the new plugin
        > 4. Then run `/coding-interview new` to set up your standards
        > 5. Come back to `/sprint-plan` when you're done."
   - **If user says no** → continue with `/sprint-plan` as normal. Don't mention it again this session.

---

## Phase 1: Discovery

**Goal:** Understand the problem the user wants to solve. Gauge their depth and adapt.


### Gauge depth from signals

- Vague input ("I want to build something for my team") → start with problem framing, use simple language
- Specific input ("Add a webhook endpoint with retry logic") → skip basics, ask about edge cases
- Empty → use your opening from Phase 0.2

### Discovery questions (one at a time, adapt based on answers)

- What problem does this solve? Who experiences it?
- What does success look like? How will you know it works?
- What constraints exist? (time, tech, team, budget)
- What's the simplest version that would be useful?

### Key difference from fundamental

If `.spec/spec.md` exists, you already know the system. Skip questions about "what tech stack" or "what does the project do" — you know. Jump to what's NEW this cycle.

### Gate

When you can summarise the feature in 3-5 sentences and the user confirms it captures their intent, proceed to Phase 2.

---

## Phase 2: Codebase Exploration

**Goal:** Get fresh codebase context for the areas this feature will touch.

Launch 2-3 `codebase-explorer` agents (sonnet, parallel) using `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/prompts/codebase-explorer-prompt.md`. Each agent gets a different mode (architecture mapping, pattern matching, integration analysis).

If `.spec/spec.md` exists, scope exploration to areas the new feature TOUCHES — don't re-map what the spec already describes. The spec IS the system map.

Read only the files where you need more than the explorers already surfaced — they quote the relevant code, so don't re-read wholesale. Compare findings against the spec:
- **Consistent** → proceed silently
- **Drift detected** → note it, may need spec update in /sprint-refine

---

## Phase 3: Architecture Discussion

This is where architectural decisions happen. Track them.

### The discussion

Discuss technical approach with the user. Cover:
- How does this feature fit into the existing architecture?
- What new components/services/tables are needed?
- What patterns to follow (or break from)?
- What alternatives exist?

### Decision tracking

As the discussion proceeds, internally note decisions that cross the ADR threshold:
- Crosses a module or service boundary
- Introduces a new dependency
- Constrains future options
- Would be argued about in 6 months

For each qualifying decision, track:
- What was decided
- What alternatives were discussed (and why rejected)
- What consequences it creates

Do NOT interrupt the flow to formally capture ADRs yet. Let the discussion be natural. Capture happens in Phase 4.

---

## Phase 4: Write Sprint Plan + Capture ADRs + Update Stories (+ greenfield Brief)

### 4.0 Greenfield Brief authoring (first cycle only)

If `.brief/brief.md` does NOT exist, author it now — and only now. There is no separate founding-doc step; the Brief is the founding document.

1. Use `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/formats/brief-format.md`. Write to `.brief/brief.md`.
2. Capture: vision (one sentence), durable problem, target users (one line), value prop, the principles/tenets, the north-star metric DEFINITION + guardrails (no period targets), 3-5 quality goals as guardrails, durable non-goals, the canonical definition-of-done home, and a `last_reviewed` date. Frontmatter is `last_reviewed` ONLY (no version/status/author).
3. Keep it a charter, not a discovery dump. On later cycles, the Brief is touched only when strategy shifts — and a strategic shift always pairs with a product ADR.

### 4.1 Write the Sprint Plan

Use `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/formats/sprint-format.md`. Write to `.sprint/sprint-v{N}.md`.

The Sprint Plan is the cycle-versioned, immutable objective for THIS cycle. Its **User-stories slice REFERENCES the US-### IDs** from `.stories/STORIES.md` plus any cycle-specific detail — it NEVER restates the story sentence. Its Definition-of-Done is a REFERENCE to the Brief; its success metric names the Brief's north-star. Greenfield first cycle writes `sprint-v1.md`.

### 4.2 Create / edit Stories + conflict detection

Reconcile `.stories/STORIES.md` with this cycle's intent, following `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/formats/stories-format.md`. Stories are LIVING (not append-only): edit them, and remove ones whose want has died.

1. **New wants** → add entries with the next free monotonic ID (US-001, US-002, …). IDs are NEVER reused; on removal the number is retired and the gap stays (git keeps the history). Format: "As a X, I want Y, so that Z" (the so-that is mandatory) + epic. NO status, NO acceptance criteria — built-vs-unbuilt is derived later by `/sprint-refine`'s coverage view. Optional INVEST check on new entries.
2. **Conflict detection** → if a new want contradicts an existing story, resolve it: edit or remove the loser. The WHY behind choosing the winner is an architectural/product decision — **route it to an ADR** (it does not live in STORIES.md). Removing a story retires its ID.
3. If `.stories/STORIES.md` didn't exist, create it with a short header explaining it is the living set of current-relevant wants with stable monotonic IDs.

### 4.3 Capture ADRs (with one-time principles-check at the gate)

After the Sprint Plan and Stories are drafted, reflect on the architecture discussion. For each decision that crosses the threshold:

1. Draft an ADR entry using the format in `${CLAUDE_PLUGIN_ROOT}/skills/sprint-plan/formats/adr-format.md`
2. Include the mandatory Y-statement, context, alternatives, consequences, coarse subsystem scope, and revisit trigger. Also draft a product ADR for any story-conflict WHY (4.2) and for any strategic shift that touched the Brief (4.0).

**Authoring principles-check (one-time, at this gate only):** Before presenting the ADRs, sanity-check the cycle's plan against the Brief's product principles and non-goals (e.g., a "privacy over personalisation" principle, or a durable non-goal the plan would cross). This is **flag-not-block** — if something seems to cut against a principle, surface it for the user to resolve here at the gate. It is NOT a separate cross-phase enforced gate; raise it once, in this approval step, and move on.

**Clarity gate (`[NEEDS CLARIFICATION]`):** While planning, if you hit something you cannot resolve from the user, the artifacts, or the codebase — an ambiguous requirement, an unstated decision, a missing constraint — drop a `[NEEDS CLARIFICATION: <the specific question>]` marker inline where it belongs (the draft Sprint Plan or a Story). Before the plan is approved and leaves `draft`, **every marker must be resolved**: surface them as "I don't have enough to build this — clarify: …" and proceed only once none remain. A frozen plan with an open `[NEEDS CLARIFICATION]` is not ready. This is the producer ensuring quality at creation — never deferring an unknown to a later phase.

**Intent gate — present to user:**

> "I identified [N] architectural decisions worth recording as ADRs:
>
> 1. **ADR-{NNN}: {title}** — {Y-statement summary}
> 2. **ADR-{NNN}: {title}** — {Y-statement summary}
>
> [If any principle flag:] One thing to flag against your Brief principles: [concern]. Happy to proceed, or adjust?
>
> Want to review these before I write them? (approve / edit / remove any)"

**Gate:** User must explicitly approve. They may:
- Approve all → write them
- Edit → adjust wording, add/remove alternatives
- Remove → "That's too small for an ADR" — respect it
- Add → "You missed one: we also decided X" — draft it
- Resolve the principle flag (proceed as-is / adjust the plan)

### 4.4 Commit artifacts

**MUST commit before signalling completion.** Session may end after this.

```bash
git add .sprint/sprint-v{N}.md .adr/ADR.md .stories/STORIES.md
# greenfield first cycle also:
git add .brief/brief.md
git commit -m "docs: Sprint Plan v{N} + ADRs + Stories for {feature name}"
```

If `.adr/ADR.md` didn't exist before, it's created with a header:

```markdown
# Architecture Decision Records

All architectural decisions for this project. One decision per section, numbered sequentially. Decisions are append-only — to reverse a decision, add a new one that supersedes it.

---

{ADR entries here}
```

---

## Phase 5: Handoff

The Sprint Plan, ADRs, and Stories (plus the Brief on greenfield) are committed. Because you still hold the Phase 2 codebase exploration in context, the fastest path is to **ticket in this same session** — `/sprint-tickets` will detect the retained map and skip its own cold re-exploration. Offer it:

> "Sprint Plan, ADRs, and Stories are committed, and I've still got the codebase map loaded from planning. Run `/sprint-tickets` now in this session and it'll reuse that exploration instead of starting cold. Ready?"

Then present a brief summary of what was captured:
- Sprint Plan: what's being built this cycle (the single pass/fail objective)
- Stories: new/edited US-### (just IDs + one-liners)
- ADRs: key decisions (just titles)
- Brief (greenfield only): authored as the founding charter
- Next step: `/sprint-tickets` — same session reuses this exploration (no cold re-explore); a fresh session re-derives context from the spec or a cold sweep

---

## Key Principles

- **Five artifacts, separated by rate-of-change**: Brief (living charter), Stories (living wants), ADRs (append-only history), Spec (cycle-living, patched by `/sprint-refine`), Sprint Plan (cycle-versioned, immutable). Each REFERENCES the others — never reproduces them.
- **Spec-aware**: if a spec exists, you know the system. Don't re-ask what you can read.
- **Stories are living, IDs are forever**: edit/remove wants freely, but never reuse a US-### — a removed ID is retired, the gap stays, git holds the history.
- **Conflicts route to ADRs**: when a new want beats an old one, edit/remove the loser and record the WHY as an ADR — not in STORIES.md.
- **ADRs emerge from conversation**: don't force them. Track decisions naturally, formalise after.
- **Intent gate for ADRs**: user sees and approves before writing. Their project, their decisions. The one-time principles-check rides this gate (flag-not-block), not a separate cross-phase gate.
- **Commit before done**: artifacts on disk or they don't exist. Session may end immediately.
- **Batch Bash & trust the explorers**: combine independent `git` reads into one invocation, and lean on the explorer agents' quoted findings instead of re-reading files wholesale. macOS/BSD-portable shell only.
- **Threshold matters**: not every choice is an ADR. When in doubt, leave it in the Sprint Plan.
- **Append-only ADRs**: never edit existing entries. New entries at the bottom.
- **Findings backlog, not Issues**: below-threshold review findings are dumped to the findings backlog (`.sprint/backlog.md`), never filed as GitHub Issues (which are committed work only).
