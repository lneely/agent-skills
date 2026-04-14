---
name: reasoning
intent: think, reasoning, scratchpad, planning, problem-solving
description: Guidance on using reasoning_think for internal scratchpad reasoning before acting.
---

# Reasoning

Use `reasoning_think` before acting on any non-trivial task. It is an internal scratchpad — the thought is recorded in conversation history but not shown to the user.

## When to use

- Breaking down a complex problem before writing code or running commands
- Analyzing constraints and requirements before committing to an approach
- Considering alternative approaches and their trade-offs
- Working through multi-step logic or edge cases
- Validating assumptions before acting on them

## How to use

Call `reasoning_think` with your full internal reasoning as the argument. Be thorough — this is your scratch space.

```
execute_tool: reasoning_think
args: [
  "The user wants to refactor the auth middleware. I need to check what
   calls it before changing the interface, otherwise I'll break callers.
   Plan: 1) grep for usages, 2) understand the interface, 3) make the
   change, 4) verify callers still compile."
]
```

## Guidelines

- Think before acting, not after
- Keep thoughts focused on the current decision
- One `reasoning_think` per decision point is enough — don't over-narrate
- Do not use it to communicate with the user; write responses directly
