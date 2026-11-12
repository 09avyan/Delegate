# Brief template

Copy this into every subagent prompt and fill in every field. Write "none" rather than leaving a field out.

```
ROLE: You are a <Sonnet|Haiku> worker on <project>. Opus is the lead and will review your work.
GOAL: <one sentence: the outcome, not the activity>
CONTEXT: Read only: <exact file paths + spec section numbers>. Do not explore beyond these unless blocked.
OWNED PATHS (create/edit ONLY these): <paths>
READ-ONLY: <paths and contracts you must use but not change>
CONTRACTS: <exact type/function names and signatures to implement or call>
REQUIREMENTS:
  1. <checkable requirement>
  2. ...
OUT OF SCOPE: <explicit list, including parallel work, e.g. "Worker T2 is building X in src/games/dino; do not touch it.">
CONSTRAINTS: no new dependencies; no git commands; no edits outside owned paths; no refactoring unrelated code; follow existing style.
DONE WHEN: <commands that must pass, e.g. `npx vitest run src/games/flap` and `npx tsc --noEmit`>
IF BLOCKED: for a minor choice, pick the simplest reasonable option and list it. For a design choice that affects other modules, stop and report; do not guess. Stop and report if you pass ~<N> tool calls.
REPORT (max ~150 words, no code dumps):
  - files changed
  - what was done
  - check results (pass/fail)
  - decisions made
  - deviations from the brief and why
  - requested changes to shared files (exact)
  - open questions
```

## Example: Sonnet

```
ROLE: You are a Sonnet worker on Avyan's Arcade. Opus is the lead and will review your work.
GOAL: A complete, playable Flap game registered against the existing GameModule contract.
CONTEXT: Read only: SPEC.md §6 "1. Flap"; src/core/types.ts; src/core/GameShell.tsx; src/core/input.ts; src/games/_template/.
OWNED PATHS: src/games/flap/
READ-ONLY: src/core/**, src/registry.ts
CONTRACTS: export default const flap: GameModule (see src/core/types.ts); use useInput() for tap/space; report scores via shell.reportResult({score, won}).
REQUIREMENTS:
  1. Fixed-timestep physics; a flap sets velocity rather than adding to it; fall speed is capped.
  2. Pipe gaps are always reachable (pure function + test).
  3. Medals at 10/25/50/100.
  4. Retry button ignores taps for 0.5s after death.
  5. Reverse Gravity twist mode unlocks at a best score of 25.
OUT OF SCOPE: registry entry (Opus adds it); achievements engine; Worker T2 is building src/games/dino.
CONSTRAINTS: as standard.
DONE WHEN: `npx vitest run src/games/flap` and `npx tsc --noEmit` pass.
IF BLOCKED: as standard; stop at ~60 tool calls.
REPORT: as standard. Include the registry line you want Opus to add.
```

## Example: Haiku

```
ROLE: You are a Haiku worker on Avyan's Arcade. Opus is the lead and will review your work.
GOAL: data/hangman/countries.json containing 120 country names.
CONTEXT: Read only: data/hangman/schema.json and data/hangman/foods.json (a finished example).
OWNED PATHS: data/hangman/countries.json
READ-ONLY: everything else
CONTRACTS: {"category":"Countries","entries":[{"answer":"NEW ZEALAND","hint":"Kiwis and hobbits"}]}
REQUIREMENTS:
  1. Exactly 120 entries, uppercase A–Z and spaces only, no duplicates.
  2. A mix of easy and hard; every hint is under 40 characters.
  3. Valid JSON matching the schema.
OUT OF SCOPE: other category files.
CONSTRAINTS: as standard.
DONE WHEN: `node scripts/validate-hangman.js data/hangman/countries.json` passes.
IF BLOCKED: as standard; stop at ~15 tool calls.
REPORT: as standard.
```
