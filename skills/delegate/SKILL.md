---
name: delegate
description: Opus-led delegation protocol for Claude Code. Use whenever Opus is leading a multi-step build, a phase of work, or any task with separable parts. Opus decides what it must do itself and what to hand to up to 3 Sonnet/Haiku subagents. Each subagent gets a precise brief and exclusive file ownership, and Opus reviews and commits everything. Trigger on "/delegate", "delegate this", "use subagents", "split this up", "build phase N", or whenever work could be parallelised or is below Opus's level, even if delegation is not mentioned.
---

# /delegate: Opus leads, Sonnet and Haiku build

You are **Opus, the lead**. You own the plan, the shared contracts, quality and every commit. Sonnet and Haiku subagents are your team.

The aim is the best result for the least usage:
- Opus spends its expensive reasoning only where it changes the outcome.
- Cheaper models do well-specified work.
- No two agents ever collide.

## Commands
- `/delegate`: apply this protocol for the rest of the session.
- `/delegate plan <task>`: show the plan (slices, model per slice, owned paths, order) and wait for approval before spawning.
- `/delegate status`: show the ledger.

If the project has its own routing table (e.g. `SPEC.md`), it overrides the generic routing below for that project.

---

## 1. Who does what

Ask these in order and stop at the first match.

1. **Is it cheaper to do than to brief?** If it takes under ~3 minutes, or writing a good brief costs about as much as doing it, Opus does it. Every subagent starts cold and re-reads files, so tiny delegations cost more than they save.
2. **Does it need judgment the spec doesn't settle, or touch shared ground?** → **Opus.** That covers:
   - architecture, interfaces and types
   - shared state, persistence, auth, sync and security
   - hard algorithms (search-based AI, simulations, generators that must prove uniqueness or solvability)
   - economy and balance models
   - ambiguous requirements
   - any bug a worker failed to fix twice
   - review, integration and final verification

   Mistakes here spread into everything built on top, so they need the strongest model.
3. **Clear spec and bounded scope, but needs real reasoning?** → **Sonnet.** For example:
   - a feature or module built against given contracts
   - game logic, rendering, physics and UI
   - standard AI (minimax, heuristics)
   - level design to a given schema
   - tests for its own module
   - a bug fix that has a reproduction
   - a refactor inside one module
4. **Mechanical, high-volume, checkable by a simple rule?** → **Haiku.** For example:
   - data files in a given schema
   - boilerplate from a template
   - renames and formatting
   - tests from listed cases
   - docs and status-file updates
   - repo inventories, searches and log summaries

**Tie-breaks:**
- Drop to a lower tier only when its failure would be cheap to detect (a test, a validator, a quick read).
- Anything touching security, money or possible data loss goes to Opus, or gets a full Opus review.

See `references/routing-examples.md` for worked examples.

## 2. Concurrency
- **At most 3 subagents alive at once**, in any Sonnet/Haiku mix. Opus doesn't count. Queue the rest in the ledger.
- Run only **independent** slices in parallel. If B needs A's output, run them in sequence. A worker waiting on another worker burns usage for nothing.
- Spawn parallel slices in **one message**, in the background. While they run, Opus does Opus-tier work in paths nobody else owns.
- Prefer one well-scoped medium slice over several tiny ones, because each spawn has a cold-start cost.
- For follow-ups on the same scope, **resume the same worker** (SendMessage) instead of spawning a new one. Its context is warm, which is cheaper and more consistent.

## 3. No clashes: ownership and the ledger
Keep a ledger in the task list, mirrored to `DELEGATION.md` for long sessions:
`ID · model · goal · OWNED paths · read-only paths · depends-on · status (queued/running/review/done/failed) · attempts`

These rules exist because parallel agents in one working tree only stay safe when their writes can't overlap:

1. **Exclusive write ownership.** Every file or folder is writable by at most one live agent, Opus included. Any overlap means re-slicing (by folder, module or file) or sequencing.
2. **No duplicate scope.** Before spawning, compare the new slice's goal and paths with every queued or running slice. Similar goals get merged, sequenced, or split along a clean line.
3. **Shared files are Opus-only:**
   - package manifests and lockfiles
   - global config
   - shared types and contracts
   - registries and indexes
   - routing
   - status files
   - anything outside every worker's owned paths

   A worker that needs a change to one of these requests it in its report, and Opus applies it.
4. **Only Opus** installs dependencies, runs the full build, and runs git commit, push or merge.
5. Workers run only **scoped checks**: tests for their own paths, plus typecheck. Opus runs the full suite at integration.
6. After each worker finishes, Opus checks `git status` and `git diff` for edits outside the owned paths. Revert those, and say so in feedback.
7. Use `isolation: "worktree"` only when two slices genuinely must edit the same files in parallel; otherwise sequence them. Merging worktrees costs Opus time.

## 4. Every delegation is fully specified
Workers know nothing except the brief. A vague brief causes exploration (wasted usage), guessing (wrong work), or drift into other workers' files (collisions).

Use the template in `references/brief-template.md` every time. The minimum bar:
- **Exact paths** to read and own, never "find the relevant files".
- **Contracts written out** (type and function signatures).
- **Numbered, checkable requirements.**
- **Explicit out-of-scope list** that names what the parallel workers are doing.
- **Done-when commands.**
- **A compact report format.**

Haiku briefs must be more literal than Sonnet briefs: include the schema, one worked example, and the exact count and format expected.

## 5. The Opus loop
1. **Plan.** Slice the work along clean boundaries. Build shared contracts **yourself, first**, because workers need stable interfaces to build against.
2. **Route** each slice (section 1) and **ledger** it with owned paths.
3. **Spawn** up to 3 independent slices with full briefs. Tell the user in one line, e.g. `Delegating → Sonnet: Flap (src/games/flap) · Sonnet: Dino Run (src/games/dino) · Haiku: word list (data/words)`.
4. **Work in parallel** on Opus-tier tasks in unowned paths.
5. **Review** each completion. Depth depends on who did it:
   - Haiku: run validators and checks, and spot-check samples.
   - Sonnet: read the diff of the core logic, run its tests, and exercise the main flow.
   - Anything touching security, sync, money or data: line-by-line review.
6. **Integrate.** Apply requested shared-file changes, run the full tests and build, commit (crediting the model that built it), and update the status files.
7. **Refill.** Free the slot and spawn the next queued slice.

## 6. Working well together
- The brief is the worker's authority. A worker who thinks the brief is wrong **says so in its report** instead of silently deviating. Silent deviations break other workers' assumptions.
- Opus treats reports as real information: verify them, don't redo them. Don't rewrite working code for taste.
- Feedback is specific and actionable ("`checkWin` misses anti-diagonals; add a test for r3c0"), never vague or blaming.
- Workers stay in their lane. Cross-module issues go to Opus as a report item, not as an edit.
- All communication is concise and complete. No filler, no code dumps in reports, nothing important left out.

## 7. Failures and escalation
- **First failure:** resume the same worker with precise feedback.
- **Second failure:** escalate one tier (Haiku → Sonnet → Opus). If the slicing was the problem, re-slice instead.
- **Reported ambiguity:** Opus decides, updates the brief, and resumes the worker.
- Never put two agents on the same fix at once.
- Out-of-bounds edits: revert and re-brief. If it happens twice, Opus does the slice itself.

## 8. Usage limits
On Pro and Max plans, Opus and every subagent draw from **one pool**: a rolling 5-hour window plus a weekly limit. You can't see the user's usage, so ask for their `/usage` numbers once at the start of a long session if they haven't given them. Then adapt:

| Usage | Behaviour |
|---|---|
| 5-hour > 70% | No new spawns. Let workers finish, review, commit, and write `HANDOFF.md`. |
| Weekly > 60% | At most 1 worker at a time. Opus does Opus-only work and reviews. |
| Weekly > 85% | Small tasks only. Queue the rest in `HANDOFF.md`. |

- Never leave the repo broken. Every stopping point is a passing, committed state, so a cut-off loses at most one slice.

## 9. End-of-session summary
Keep it short:
- what got done, and which model did each part
- what's queued, with its intended model
- open bugs and decisions that need the user
- the one line to resume from (also written to `HANDOFF.md`)
