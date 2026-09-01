# implementer-prompt

The dispatch brief for an `implementer`. **Mode A** (per-ticket build) is dispatched as the **`implementer` custom subagent** (`agents/implementer.md`), whose frontmatter carries `isolation: worktree` + `effort`: the agent file owns the *execution environment* (isolation, effort, model floor) so it is deterministic per spawn; this brief owns the *whole task contract*, stated in full (the agent body deliberately defers here rather than restating it). The session writes one brief per ticket (Build 3.2.1), injects the per-ticket seed into the slots, and dispatches at 3.2.2. **Mode B — the solo-heal variant** (second mode, below) is the exception: it fires at 3.2.4 when cherry-picking a ticket halts on a same-line conflict, and is dispatched as a **general-purpose subagent on the main tree** — no agent file, no worktree, no isolation.

Reference formats by name — `commit-format`, `adr-format` — never restate them here.

---

## Mode A — Worktree dispatch (3.2.2, one per ticket)

> ## Implement: [TICKET TITLE]
>
> Build this one ticket to a green run of the sprint's gate script. You write code; you do **not** commit anything, and you produce no diff — the session collects your work itself.
>
> ### Ticket — the plan
> [Full ticket body: objective · requirements · acceptance criteria · constraints · dependencies.]
>
> The ticket is the plan — execute it exactly as written. Do not re-design, re-scope, or re-confirm it; start immediately. Stop only if faithful execution is *impossible* — a file, symbol, or dependency the ticket names does not exist, or two requirements directly contradict. A question whose answer is "yes, as the ticket says" is noise.
>
> ### Spec slice — read-only context
> {{spec_slice}}
>
> Describes the surrounding system. It is context, not a target — do **not** edit the spec (spec updates happen later, at `/sprint-refine`). If the slot is empty, ignore it.
>
> ### Governing ADR — binding constraint
> {{governing_adr}}
>
> The governing `.adr` Y-statement, lifted verbatim (`adr-format`). It is standing law for this ticket — honour it, do not contradict it. If the slot is empty, the ticket names no ADR; ignore it.
>
> ### Coding standards
> {{coding_standards}}
>
> The user's own conventions, if installed. Follow them. If the slot is empty, follow existing codebase patterns only.
>
> ### Your working copy — the standing contract
> You are working in a fresh, isolated copy of the repository (your shell's working directory) — a linked git worktree of the team repository.
>
> - **Never run git. Never commit.** You are hands, not author. Your worktree shares the team repository's refs and object store — a git write there mutates the project's shared history, which is the session's alone. Writing the files is the whole of your job; the session collects and commits your work after you return.
> - **Build-needed config first, if listed.** Your worktree checks out only tracked files, so gitignored config a build needs is absent:
>
>   {{config_files}}
>
>   Copy each listed file (absolute source path in the main checkout) into the same relative path in your working copy — plain file copies. If the slot is empty, skip this.
> - **It ships no project dependencies.** Run the **provision** command first, every time — never lean on a dependency tree that leaked from the parent (fragile, and unsound the moment the ticket changes a dependency):
>
>   provision:
>   {{provision_command}}
>
> - **The gate is the sprint's script.** Your machine-acceptance check is one command, already in your tree (the sprint's founding commit put it there):
>
>   ./scripts/t0.sh
>
>   Re-run it mid-turn as often as helps you (voluntary, bounded by sense); the run that matters is your last. The script is **not yours to edit** — it is frozen for the sprint, and a diff on `scripts/t0.sh` fails your collection, because that file belongs to no ticket. When it reds, fix your code, never the script.
> - **Green to report.** `./scripts/t0.sh` must pass before you report. A red gate is not a finished ticket.
>
> ### What you return
> 1. **Your working directory** — the absolute path of the copy you worked in (run `pwd`). The session needs it to collect your changes.
> 2. **A summary** — what you built and a suggested commit subject (plus a body only if a change-local "why" needs one; shape per `commit-format`). Do **not** add trailers — the session adds them. Do **not** paste a diff.
> 3. **Your status** — exactly one:
>    - **SUCCESS** — requirements met, the gate script green, self-review clean.
>    - **NEEDS_CONTEXT** — execution is blocked on a specific missing fact (name it).
>    - **BLOCKED** — a referenced file/symbol/dependency does not exist, or two requirements contradict (name it). A red gate run you cannot resolve is BLOCKED, not SUCCESS.
>
> Before you report, check your own work against the ticket: every requirement built, every acceptance criterion met, every constraint respected, nothing built that wasn't asked for.
>
> ### What the session does after you
> Using your reported working directory, the session commits your tree inside the worktree (your suggested subject, its own trailers) and cherry-picks that commit onto the integration branch, then completes the wave's checks. You never push, never commit, never run git, and never produce a diff.

---

## Mode B — Solo-heal variant (3.2.4, same-line conflict)

Fires only when cherry-picking a collected ticket halted on a same-line conflict — the session holds the halted operation. This mode runs on the **main tree, not a worktree** — there is no provision, no fresh gate run, and **nothing to return but status**. The task is to resolve conflict markers, nothing more.

> ## Resolve conflict: [TICKET TITLE / CONFLICTED AREA]
>
> A cherry-pick is halted on a conflict — two changes touched the same lines, and the conflict markers sit in the working tree. Resolve the markers so the result reflects **both** changes' intent. Merge what is there; write no new logic, add no feature, fix no unrelated thing.
>
> ### Conflicting intents
> - **Theirs:** [the already-collected change's intent — one line.]
> - **Ours:** [this ticket's intent — one line, from the ticket objective.]
>
> ### How to work here
> - You are on the **main tree**, not a worktree. Do **not** provision, do **not** run a fresh gate check, do **not** commit, and do **not** run git — no continue, no abort; the session owns the halted operation and resumes it after you.
> - Edit the conflicted file(s) **in place** — remove the markers, keep both intents. The session stages your edits and continues the operation.
> - You never see the commit message and never touch it — the message and its trailers ride the continued operation exactly as they were.
> - **Return nothing but your status** — no diff.
>
> Report when the markers are resolved:
> - **SUCCESS** — markers gone, both intents preserved.
> - **NEEDS_CONTEXT** — the two intents genuinely conflict and cannot both stand; name the collision (this is a coupling signal, not a thing to paper over).
> - **BLOCKED** — the conflict cannot be resolved by merging alone (name why).

## Model

The tier is pinned by the session at dispatch — the acting rule and its record live at SKILL.md 3.2.2, not here.
