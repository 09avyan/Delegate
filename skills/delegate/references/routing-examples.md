# Routing examples

| Task | Goes to | Why |
|---|---|---|
| Design the game registry, shared types and save-file format | Opus | Every later slice depends on it, so a mistake spreads everywhere |
| Auth + sync serverless function holding secrets | Opus | Security and data-loss risk |
| Chess AI (alpha-beta, quiescence, time control) | Opus | Hard algorithm with subtle bugs |
| Sudoku generator with a uniqueness proof | Opus | Correctness proof needed |
| Build Flap from its spec section | Sonnet | Clear spec, bounded folder, real reasoning |
| Connect 4 minimax AI | Sonnet | Standard, well-known algorithm |
| Fix "pieces overlap after line clear", with repro steps | Sonnet | Bounded bug with a known reproduction |
| 20 rage-game levels in a given JSON schema | Sonnet | Creative, but follows a schema; verifiable by playing |
| 120 Hangman words in a given schema | Haiku | Mechanical; a validator checks it |
| Update README and GAMES.md after a phase | Haiku | Docs from known facts |
| Find every place `localStorage` is used | Haiku, or Opus with grep | One grep is cheaper than a spawn: Opus does it |
| Change the Flap pipe colour | Opus (do it) | Faster than writing a brief |
| Same bug, second failed fix by Sonnet | Opus | Escalation rule |

## Slicing patterns
- **By folder:** one game per worker (`src/games/<slug>/`). This is the cleanest pattern.
- **By layer:** Opus writes the contract, then Sonnet builds the logic and UI against it, then Haiku writes the data.
- **Never by "half a file":** if two slices need the same file, sequence them.

## Parallel set examples (max 3)
- Sonnet: Flap · Sonnet: Dino Run · Haiku: word list → independent folders. ✅
- Sonnet: Chess UI · Opus: chess engine in `src/games/chess/engine/` (UI reads the engine interface only) → fine if the interface is written first. ✅
- Sonnet: Ludo · Sonnet: "fix shared input layer" → conflict, because Ludo uses the input layer. Sequence them: input fix first. ❌
