# Hold My Beer 🍺

**Filesystem-backed working memory for AI agents.**

Three files that give any agent with filesystem access a new memory tier — cheaper than context tokens, more natural than semantic search, and survives session resets. Works with any framework: Letta, Claude Code, Cursor, Windsurf, Copilot, Aider, or anything that can read and write files.

## The Problem

Out of the box, Letta agents have three places to keep state: the context window (raw conversation, expensive, dies at compaction), memory blocks (Letta's claim to fame — persistent, always in context, but size-limited), and archival memory (unlimited storage, but requires API calls and semantic search to retrieve). Files are also available but read-only from the agent's perspective in standard configurations.

Yes, you can extend all of this with custom tools, MCP servers, external databases, and whatever else you want to bolt on — but we're talking about what ships with every Letta agent by default.

In that default configuration, there's no good place for ephemeral working state — the variable values, intermediate results, and scratch notes you need during a task but don't want permanently in your memory blocks. Memory blocks are for identity and long-term knowledge, not for holding a list of file paths you're processing right now. But the mechanism for a working cache already exists: **the same system that loads and unloads skills can serve as a read/write scratchpad.**

## The Solution

Any agent system that can read and write files already has everything it needs. Reading a file injects its contents into context. Writing a file persists state to disk. Put those together and you have a **read/write cache backed by the filesystem** — using capabilities that are already part of any agentic platform with file access. No new tools, no external services. Free to store, costs tokens only when loaded, and survives everything.

```
┌──────────────────────────────────────────────────────────────────┐
│  Memory Tier        Cost              Persistence    Read-Write  │
├──────────────────────────────────────────────────────────────────┤
│  Context window     Always on         Dies at comp.  Read-only   │
│  Memory blocks      Always on         Survives comp. Read-write  │
│  ★ Beer cache       Free until loaded Survives all   Read-write  │
│  Archival memory    Search required   Permanent      Write-easy  │
│                                                      Read-costly │
└──────────────────────────────────────────────────────────────────┘
```

Beer cache fills the gap between memory blocks (always consuming context tokens, size-limited, meant for permanent knowledge) and archival memory (requires a search to retrieve anything). It's the working desk. Memory blocks are the whiteboard on the wall. Archival is the filing cabinet.

## What's In The Box

### `hold-my-beer/`
Emergency compaction defense. When context is critically full, the agent loads this skill, saves a structured recovery snapshot to archival memory, verifies it landed, and unloads. The agent saves what matters because **the agent knows what matters** — not a blind dump of recent messages.

### `beer-cache/`
The cache file itself. Starts empty. The agent writes working state here with standard file tools, loads it into context with `Skill("beer-cache")` when needed, and clears it when done. Think of it as a scratchpad that lives on disk instead of in the context window.

### `beer-on-tap/`
Instructions for using the cache system. Loaded when the agent needs to understand how beer-cache works — like after compaction when the agent's been reset but the cache file is still sitting on disk with useful state in it.

## Platform Compatibility

This isn't a Letta feature. It's a filesystem feature.

The only requirements are: **(1)** the agent can read a file, **(2)** the agent can write a file, and **(3)** file contents end up in the agent's context when read. That's every coding agent framework: Claude Code, Cursor, Windsurf, Copilot, Aider, Letta Code — anything with filesystem access.

The SKILL.md format and `Skill()` loader are Letta conventions that make discovery and loading ergonomic, but the underlying mechanism is just "read a markdown file." On other platforms, replace `Skill("beer-cache")` with whatever reads a file into context — `cat`, `Read`, `include`, or just asking the agent to read it.

The pattern is universal. The files are just files.

## Installation

### Letta Code / Letta Agents

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

### Any Other Agent Framework

Put the files wherever your agent can read and write. Point your agent at `beer-on-tap/SKILL.md` for instructions. That's it.

No configuration. No API keys. No hooks to wire up. No platform lock-in.

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

## Prior Art: TodoWrite

If this pattern looks familiar, it should. Letta Code's built-in `TodoWrite` tool does the same thing — it writes structured state to a temporary file on disk and injects it into the agent's context. The todo list you see in Letta Code sessions is a file with a random name that gets created, updated, and read back using exactly this mechanism: filesystem-backed state, injected on demand.

We didn't invent this pattern. We arrived at it independently from the compaction defense angle and then recognized it as the same architecture Letta's team already uses for task tracking. What we're adding is:

- **General-purpose** — not locked to a todo schema. Any structure, any content.
- **Agent-controlled format** — the agent decides what the cache looks like based on the task.
- **Explicit and transparent** — the agent knows where the file is and manages it with standard tools, not a specialized API.
- **Multiple caches possible** — you could have `beer-cache-db/`, `beer-cache-deploy/`, etc.
- **Compaction defense integration** — designed to work with hold-my-beer for pre-compaction saves.

The validation: if Letta's own team built TodoWrite on this architecture, the pattern is sound.

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
