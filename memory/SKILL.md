---
name: memory
intent: remember, recall, memory, persist, forget
description: Guidance on using memory_remember and memory_recall to persist and retrieve information across sessions.
---

# Memory

Use `memory_remember` to persist facts that would otherwise be lost when the session ends. Use `memory_recall` to search for relevant context at the start of a new topic or when the user references something you may have stored before.

## When to remember

- The user states a preference, constraint, or decision that affects future work
- You learn a non-obvious fact about the codebase, system, or environment
- A debugging session surfaces a root cause worth keeping
- The user explicitly asks you to remember something

Do not remember ephemeral task state, things derivable from the current codebase, or things already in git history.

## When to recall

- The user references something that sounds like prior context ("like last time", "the one we discussed", "what we decided about X")
- Starting work in an area where stored context might exist
- The user asks what you know about a topic

## How to use

```
execute_tool: memory_remember
args: ["<title>", "<tags_csv>", "<body>"]
```

- `title`: short noun phrase describing the memory
- `tags_csv`: comma-separated tags; pick specific and reusable terms
- `body`: the fact or context worth keeping, written so it stands alone

```
execute_tool: memory_recall
args: ["<query>"]
```

- `query`: keyword, tag, or phrase — searches both filenames and body content
