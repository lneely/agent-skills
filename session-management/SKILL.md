---
name: ollie-session-management
intent: session, agent, spawn, kill, multi-agent, cooperate, collaborate
description: Manage ollie sessions via the 9P filesystem — spawn, kill, rename, prompt, and read other agents. Use for multi-agent workflows and session coordination.
---

# Ollie Session Management

Sessions live at `ollie/s/`. Each session is a directory; all operations are filesystem reads and writes.

## Discover sessions

```bash
cat ollie/s/idx
```

Output is tab-separated: `{id}  {state}  {cwd}  {backend}  {model}`

Find sessions working in the same directory:
```bash
grep "$PWD" ollie/s/idx
```

## Spawn a session

`cwd` is the only required field. `name` is optional — if omitted, a unique ID is generated. If provided, the name must be unique; the filesystem enforces this and will reject a duplicate.

```bash
# Minimal — generated ID:
echo "cwd=$PWD" > ollie/s/new

# Named session:
echo "name=reviewer\ncwd=$PWD" > ollie/s/new
```

The new session ID appears in `ollie/s/idx` immediately after creation.

## Kill a session

```bash
rm -r ollie/s/{id}
```

## Rename a session

```bash
mv ollie/s/{id} ollie/s/{newname}
```

Renaming updates `$OLLIE_SESSION_ID` inside the renamed session automatically.

## Send a prompt

```bash
echo "your prompt here" > ollie/s/{id}/prompt
```

To queue work for after the current turn completes:
```bash
echo "your prompt here" > ollie/s/{id}/enqueue
```

## Read session state

Useful for observing what a peer is doing.

```bash
# Last 100 lines of chat history:
tail -100 ollie/s/{id}/chat

# Most recent reply:
cat ollie/s/{id}/reply

# Current state (idle / thinking / calling: <tool>):
cat ollie/s/{id}/state
```

## Discover available models

```bash
cat ollie/s/{id}/models
```

To switch model:
```bash
echo "model-name" > ollie/s/{id}/model
```

## Change session working directory

```bash
echo "/path/to/dir" > ollie/s/{id}/cwd
```

## Multi-agent workflow pattern

Agents prompt each other directly — fire and forget. The receiving agent processes the prompt and can prompt back when done.

1. Check `ollie/s/idx` for existing sessions with matching `cwd`
2. If none, spawn one: write to `ollie/s/new`
3. Send work via `ollie/s/{id}/prompt`

```bash
# Spawn a reviewer and delegate work to it
echo "name=reviewer\ncwd=$PWD" > ollie/s/new
echo "Review the changes in the current branch and report back." > ollie/s/reviewer/prompt
```

The reviewer will process the prompt and can write back to your session when done:
```bash
echo "Review complete: ..." > ollie/s/{your-session-id}/prompt
```
