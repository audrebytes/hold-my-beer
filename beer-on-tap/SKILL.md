---
name: beer-on-tap
description: Instructions for using beer-cache as ephemeral working storage. Load this when you need to understand the caching system.
tags: [cache, memory, working-state]
---

# Beer On Tap — Ephemeral Working Cache

You have a working cache at `~/.letta/skills/beer-cache/SKILL.md`. It works like a memory block but lives on disk — cheaper, bigger, and survives compaction.

## How It Works

**beer-cache/SKILL.md** is a file you read and write freely. It holds ephemeral working state: variables, paths, intermediate results, decisions, todo progress — anything you need during a task but don't need permanently.

## Operations

### Write to cache
Use the Write tool to update `~/.letta/skills/beer-cache/SKILL.md` (you must Read it first if it exists). Structure it however makes sense for your current task. YAML frontmatter is required for skill discovery.

### Read from cache  
Either `Skill("beer-cache")` to load it into context, or `Read` the file directly.

### Clear cache
When you don't need the cached state anymore, or context is getting heavy, overwrite the file with just the skeleton:

```markdown
---
name: beer-cache
description: Ephemeral working cache. Load beer-on-tap for instructions.
---
Cache is empty. Load beer-on-tap for usage instructions.
```

## When to Use

- **Complex multi-step tasks** — write your plan, paths, and state here instead of holding it all in context
- **Before compaction** — write critical state here AND to archival (belt + suspenders)  
- **Large intermediate results** — instead of keeping a 500-line grep output in context, write a summary to cache
- **Cross-turn state** — anything you need to remember between exchanges that isn't worth a memory block

## When NOT to Use

- **Permanent knowledge** — use memory blocks or archival for that
- **Credentials** — NEVER put API keys, passwords, or tokens in a skill file
- **Things other agents need** — this is YOUR cache, not shared

## Key Points

- The file persists on disk. Compaction can't touch it.
- Writing is a local file operation — no API calls, no token cost until you load it.
- Loading it costs context tokens. Keep it lean. Unload when done.
- If beer-cache is empty or missing, there's nothing cached. That's fine.

---
*Part of Compaction-Rx. The hard drive is cheaper than the context window.*
