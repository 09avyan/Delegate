# Delegate

A Claude Code skill: **Opus leads, Sonnet and Haiku build.**

Opus (high effort) plans the work, does the parts that need judgment, and delegates the rest to up to 3 Sonnet/Haiku subagents at a time. Each subagent gets:
- a precise brief
- files it alone may edit (so no two agents clash)
- Opus's review before anything is committed

The skill also adapts to Pro/Max usage limits (the 5-hour window and the weekly cap).

## Install

**Per project (recommended):** copy the skill into your repo:
```bash
mkdir -p .claude/skills && cp -r skills/delegate .claude/skills/
```

**For all your projects:**
```bash
mkdir -p ~/.claude/skills && cp -r skills/delegate ~/.claude/skills/
```

**As a plugin:** this repo is also a Claude Code plugin (`.claude-plugin/plugin.json`).

## Use
| Command | What it does |
|---|---|
| `/delegate` | Turn on the protocol for this session |
| `/delegate plan <task>` | Show the delegation plan and wait for your OK |
| `/delegate status` | Show who is doing what |

Recommended session: **Opus, high effort, ultracode off.** Ultracode starts its own multi-agent workflows, which would duplicate this skill's delegation.

## Files
```
skills/delegate/
├── SKILL.md                      # the protocol
└── references/
    ├── brief-template.md         # brief template + Sonnet/Haiku examples
    └── routing-examples.md       # who-does-what examples and slicing patterns
```
