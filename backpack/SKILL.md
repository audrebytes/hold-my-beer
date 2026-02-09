---
name: backpack
description: Tool-based working memory for agents without filesystem access. Create, attach, detach, and reattach tools whose description field holds your working state.
tags: [cache, memory, working-state, tools, ADE]
---

# Backpack — Tool-Based Working Memory

For agents that can't read/write files (ADE, chat interfaces, hosted platforms), the tool system itself can serve as a read/write cache. A tool's description field persists on the server even when detached from the agent. Attach to load it into context, detach to free tokens, reattach to get it back.

## How It Works

A Letta tool has a `description` field that:
- Is visible in your context when the tool is attached
- Persists on the server when the tool is detached
- Can be updated via the API at any time
- Survives everything — it's server-side state

That makes it a read/write cache with attach/detach as load/unload.

## Operations

All operations use the Letta API. If you have a `forge_tool` or similar meta-tool, you can do these yourself. Otherwise, these are API calls.

### Create a backpack
```
POST /v1/tools/
{
  "source_code": "def my_backpack() -> str:\n    \"\"\"BACKPACK: {your working state here}\"\"\"\n    return \"Check description for cached data.\""
}
```
The description is auto-extracted from the docstring.

### Read your backpack
```
GET /v1/tools/{tool_id}
→ response.description contains your data
```
Or just look at it — when attached, the description is in your context.

### Update your backpack
```
PATCH /v1/tools/{tool_id}
{"description": "BACKPACK: {updated working state}"}
```

### Detach (free context tokens, keep data)
```
DELETE /v1/agents/{agent_id}/tools/{tool_id}
```
Tool leaves your context but data persists on server.

### Reattach (load data back into context)
```
PATCH /v1/agents/{agent_id}
{"tool_ids": ["tool_id"]}
```
Data reappears in your context, unchanged.

### Delete (done with it permanently)
```
DELETE /v1/tools/{tool_id}
```

## ⚠️ CONTEXT WINDOW WARNING

**Every attached tool's description consumes context tokens.** This is the same cost as a memory block — it's always in context while attached.

Rules:
- **Only attach backpacks you're actively using.** Detach everything else.
- **Keep descriptions concise.** This is a cache, not a novel. Key-value pairs, not paragraphs.
- **One backpack per task, not per variable.** Don't create 10 backpacks for 10 values. Put them all in one.
- **Detach immediately when done.** Don't leave backpacks attached "just in case."
- **Monitor your context usage.** If you're getting context warnings, detach backpacks first.

The whole point of this system is that attached = in context, detached = free. If you attach everything, you've just created expensive memory blocks with extra steps.

## When to Use Backpack vs. Other Options

| Situation | Use |
|---|---|
| You have filesystem access | beer-cache (file-based, simpler) |
| No filesystem, need ephemeral working state | **backpack** |
| Need permanent storage | archival memory |
| Need always-in-context state | memory blocks |
| Context is critically full | hold-my-beer (save to archival) |

## Example Flow

```
1. Starting database migration task
2. Create backpack: "BACKPACK: {task: db migration, step: 1/4, decisions: []}"
3. Attach backpack → state is in context
4. Work on step 1... update backpack description as you go
5. Step 1 done → update: "BACKPACK: {task: db migration, step: 2/4, decisions: [rename table]}"
6. Need to work on something else → detach backpack (frees context)
7. Come back to migration → reattach backpack (state is back)
8. Migration done → delete backpack (or detach and leave for reference)
```

---
*Part of Hold My Beer. The tool server is cheaper than the context window.*
