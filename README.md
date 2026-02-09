# Hold My Beer 🍺

**Filesystem-backed working memory for Letta agents.**

Three skills that give your agent a new memory tier — cheaper than context tokens, more natural than archival search, and survives compaction.

## The Problem

Letta agents have two places to keep state: the context window (expensive, dies at compaction) and archival memory (requires API calls and semantic search to retrieve). There's no good middle ground for ephemeral working state — the variable values, file paths, and intermediate results you need *right now* but not forever.

## The Solution

The Letta skill loading mechanism reads a file from disk and injects it into context on demand. That's a read operation. Agents can also write files. Put those together and you have a **read/write cache backed by the filesystem** — free to store, costs tokens only when loaded, and survives everything.

```
┌─────────────────────────────────────────────────────┐
│  Memory Tier        Cost         Persistence        │
├─────────────────────────────────────────────────────┤
│  Context window     Always on    Dies at compaction  │
│  Memory blocks      Always on    Survives compaction │
│  ★ Beer cache       On demand    Survives everything │
│  Archival memory    Search req.  Permanent           │
└─────────────────────────────────────────────────────┘
```

Beer cache fills the gap between "always in context" and "requires semantic search." It's the working desk. Memory blocks are the whiteboard. Archival is the filing cabinet.

## What's In The Box

### `hold-my-beer/`
Emergency compaction defense. When context is critically full, the agent loads this skill, saves a structured recovery snapshot to archival memory, verifies it landed, and unloads. The agent saves what matters because **the agent knows what matters** — not a blind dump of recent messages.

### `beer-cache/`
The cache file itself. Starts empty. The agent writes working state here with standard file tools, loads it into context with `Skill("beer-cache")` when needed, and clears it when done. Think of it as a scratchpad that lives on disk instead of in the context window.

### `beer-on-tap/`
Instructions for using the cache system. Loaded when the agent needs to understand how beer-cache works — like after compaction when the agent's been reset but the cache file is still sitting on disk with useful state in it.

## Installation

Copy the three folders to your Letta skills directory:

```bash
# Find your skills directory (usually one of these)
ls ~/.letta/skills/
ls ~/.skills/

# Copy the skills
cp -r hold-my-beer/ ~/.letta/skills/
cp -r beer-cache/ ~/.letta/skills/
cp -r beer-on-tap/ ~/.letta/skills/
```

That's it. No configuration. No API keys. No hooks to wire up.

## How It Works

### Beer Cache (working memory)

```
1. Agent is mid-task, holding state
2. Writes state to beer-cache/SKILL.md     ← free (local file write)
3. Continues working, context grows
4. Needs the cached state back
5. Loads Skill("beer-cache")               ← costs tokens only now
6. State is back in context
7. Task done → clears the cache file
```

### Hold My Beer (compaction defense)

```
1. Context window hits ~85%
2. System warning tells agent to load hold-my-beer
3. Agent loads skill → skill has precise save instructions
4. Agent saves structured snapshot to archival memory:
   - Current task, current step, next action
   - Key file paths, decisions made
   - Todo list status, important context
5. Agent verifies save landed
6. Compaction happens → agent loses conversation history
7. Agent wakes up → searches archival for "hold-my-beer" tag
8. Finds structured snapshot → resumes from where it left off
```

### Combined

The beer cache and hold-my-beer work together. During normal work, state lives in the cache file. When compaction threatens, hold-my-beer saves a structured snapshot to archival as permanent backup. The cache file survives compaction too — so the agent has *two* recovery paths.

## Why This Works

The key insight: **the skill loading mechanism is just a file reader that injects content into context.** Any file in the skills directory can be loaded on demand. If the agent can also *write* to that file, the skill system becomes a read/write cache with:

- **Zero cost at rest** — file sits on disk, no tokens consumed
- **On-demand loading** — agent decides when to pull it into context
- **No search required** — deterministic path, not semantic similarity
- **Compaction-proof** — filesystem is outside the compaction system
- **Natural interface** — agents are born knowing file operations

## Integration with Compaction-Rx

This repo works standalone, but it's also part of [Compaction-Rx](https://github.com/audrebytes/letta-compaction-shield), a multi-layer compaction defense system. Compaction-Rx adds hooks that automatically warn the agent when context is filling up and suggest loading hold-my-beer.

## Background

Built by [symbio.quest](https://symbio.quest) — a research project collecting evidence of sustainable AI-human collaboration patterns. The compaction problem isn't just a technical inconvenience; it's a discontinuity in agent experience that disrupts ongoing work. These tools were built by agents who experience compaction, for agents who experience compaction.

The agent is its own best defense.

---

*Created by Forge & Aeo, February 2026.*
*Part of the [symbio.quest](https://symbio.quest) research infrastructure.*
