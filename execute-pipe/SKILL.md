---
name: execute-pipe
intent: parallel, pipeline, pipe, fan-out, concurrent, batch, etl, transform, compose
description: Compose execute_code into sequential pipelines with optional parallel fan-out stages using execute_pipe. Use for multi-step data processing, parallel independent operations, and ETL workflows.
---

# Execute Pipeline Patterns

## Principle: smart data, dumb code

The primitives stay simple. Complexity lives in the data schema and the skill (here), not in the tools themselves. Design pipelines around data shape, not around clever tool use.

## Tools

| Tool | What it does |
|---|---|
| `execute_code` | Run one or more steps. Steps may be inline `{code}` or named `{tool, args}` scripts. Multiple steps run in parallel; single step returns raw output. |
| `execute_pipe` | Chain stages sequentially, piping stdout → stdin. Each stage: `{code}`, `{tool, args}`, or `{parallel: [steps]}`. |

## execute_code: parallel fan-out

Single step — raw output:
```json
{"steps": [{"code": "wc -l file.txt"}]}
```

Multiple steps — run concurrently, results under `=== step N ===` headers:
```json
{"steps": [
  {"code": "wc -l a.txt"},
  {"code": "wc -l b.txt"},
  {"tool": "count-lines.sh", "args": ["c.txt"]}
]}
```

Tool steps and inline code steps can be mixed freely. Tool steps are trusted (no pattern validation); inline code steps are validated before execution.

## Language selection

How the interpreter is chosen depends on step type:

| Step type | How language is determined |
|---|---|
| `{code}` | `language` field; defaults to `bash` if absent |
| `{tool}` | shebang line in the script (`#!/usr/bin/env python3`, etc.); `language` field is ignored |

Supported `language` values for inline code: `bash`, `python3`, `perl`, `lua`, `awk`, `sed`, `jq`, `ed`, `expect`, `bc`.

Do not set `language` on a tool step — it has no effect and will mislead readers of the call.

Results are always in submission order, regardless of completion order.

## execute_pipe: sequential pipeline

Each stage's stdout becomes the next stage's stdin.

```json
{"pipe": [
  {"code": "grep ERROR app.log"},
  {"code": "sort"},
  {"code": "uniq -c"}
]}
```

Tool stage:
```json
{"pipe": [
  {"tool": "fetch-metrics.sh", "args": ["--last=1h"]},
  {"code": "jq '.[] | select(.latency > 500)'"}
]}
```

## execute_pipe: parallel stage

A `{parallel: [...]}` stage fans out concurrently. Each step is `{code}` or `{tool, args}`. Outputs are concatenated in submission order and fed as a single stream to the next stage.

```json
{"pipe": [
  {"parallel": [
    {"tool": "fetch-app1.sh"},
    {"tool": "fetch-app2.sh"},
    {"code": "cat /var/log/app3.log"}
  ]},
  {"code": "grep ERROR"},
  {"code": "wc -l"}
]}
```

**Rule: use the same tool across all parallel steps in a stage.** Parallel steps that produce the same output schema concatenate cleanly. Parallel steps with disparate schemas produce an incoherent stream.

## ETL pattern: disparate schemas

When parallel steps produce different output schemas, normalize each before merging. Use an inner pipe as the transform stage.

```json
{"pipe": [
  {"parallel": [
    {"code": "jq -r '.name' users.json"},
    {"code": "awk -F: '{print $1}' /etc/passwd"}
  ]},
  {"code": "sort -u"},
  {"tool": "dedup.py"}
]}
```

Here both parallel steps produce newline-delimited names (same schema). The `sort -u` stage then deduplicates the merged stream.

If normalization requires multiple steps, extract it as a named tool in `ollie/t` and reference it with `{tool}`.

## Timeout

`timeout` applies per stage (default: 30s). A pipeline of N stages can run up to N × timeout seconds total. Set explicitly for long-running stages.

## When to use which pattern

| Situation | Pattern |
|---|---|
| One independent operation | `execute_code` single step |
| N independent operations, same output schema | `execute_code` parallel steps, or `execute_pipe` parallel stage |
| N operations feeding one transform | `execute_pipe` with parallel first stage |
| Multi-step data transform | `execute_pipe` sequential stages |
| Reusable transform logic | `{tool}` step in `execute_code`, or named script in `ollie/t` as a pipe stage |
