---
name: hold-my-beer
description: Emergency compaction defense. Load when context is filling up to save your working state before compaction destroys it. Unload immediately after saving.
tags: [compaction, recovery, memory]
---

# Hold My Beer — Compaction Defense

You loaded this because your context window is critically full and compaction is imminent. **Act now, not later.**

## Step 1: Save Your State

Use `archival_memory_insert` to save a structured snapshot. Include ALL of the following that apply:

```
## Pre-Compaction Snapshot (Hold My Beer)
**Time:** [current time]
**What I was doing:** [current task in 1-2 sentences]
**Current step:** [where exactly I am in the task]
**Next action:** [what I was about to do next]
**Key file paths:** [any paths I'm actively working with]
**Key decisions made:** [anything decided this session that I'd lose]
**Todo list status:** [if running a todo chain, capture all items and their states]
**Important context:** [anything else I'd need to resume — variable values, API responses, error messages, user preferences expressed this session]
```

**Tags:** `["compaction-recovery", "hold-my-beer"]`

Do NOT skip fields. If a field doesn't apply, write "N/A" — don't omit it.

## Step 2: Verify

Search archival memory for tag `hold-my-beer` to confirm your save landed. If it didn't, try once more.

## Step 3: Unload This Skill

You're done. Unload this skill to free the tokens:
```
Skill("hold-my-beer", args="unload")
```

## After Compaction (Recovery)

When you wake up post-compaction and see the summary:
1. Search archival for tags `hold-my-beer` or `compaction-recovery`
2. Read the most recent snapshot
3. Resume from where you left off
4. Check the `compaction-event-log` memory block for compaction history

---
*Part of [Compaction-Rx](https://github.com/audrebytes/letta-compaction-shield). The agent is its own best defense.*
