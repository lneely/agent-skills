---
name: execute-pipe
intent: parallel, pipeline, pipe, fan-out, concurrent, batch, etl, transform, compose
description: Use execute_code pipeline stages for sequential data processing, parallel fan-out, and ETL workflows.
---

# Execute Pipeline Patterns

## Principle: smart data, dumb code

The primitive stays simple. Complexity lives in the data schema and the skill (here), not in the tool itself. Design pipelines around data shape, not around clever tool use.

## execute_code: the unified primitive

`execute_code` accepts a `steps` array of pipeline stages. Stages run sequentially, each stage's stdout fed to the next stage's stdin. A single stage returning raw output is the degenerate case.

Each stage is one of:
- `{code, language}` — inline code (default: bash)
- `{tool, args}` — named script from `ollie/t`; language detected from shebang
- `{parallel: [steps]}` — concurrent fan-out; outputs concatenated in submission order, fed to next stage

Tool steps and inline code steps can be mixed freely. Tool steps are trusted (no pattern validation); inline code steps are validated before execution.

## Language selection

| Step type | How language is determined |
|---|---|
| `{code}` | `language` field; defaults to `bash` if absent |
| `{tool}` | shebang line in the script; `language` field is ignored |

Supported `language` values for inline code: `bash`, `python3`, `perl`, `lua`, `awk`, `sed`, `jq`, `ed`, `expect`, `bc`.

## Single stage (degenerate case)

Raw output, no headers:
```json
{"steps": [{"code": "wc -l file.txt"}]}
```

## Sequential pipeline

Each stage's stdout becomes the next stage's stdin:
```json
{"steps": [
  {"code": "grep ERROR app.log"},
  {"code": "sort"},
  {"code": "uniq -c"}
]}
```

Tool stage:
```json
{"steps": [
  {"tool": "fetch-metrics.sh", "args": ["--last=1h"]},
  {"code": "jq '.[] | select(.latency > 500)'"}
]}
```

## Parallel stage

A `{parallel: [...]}` stage fans out concurrently. Outputs are concatenated in submission order and fed as a single stream to the next stage:

```json
{"steps": [
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

When parallel steps produce different output schemas, normalize each before merging:

```json
{"steps": [
  {"parallel": [
    {"code": "jq -r '.name' users.json"},
    {"code": "awk -F: '{print $1}' /etc/passwd"}
  ]},
  {"code": "sort -u"},
  {"tool": "dedup.py"}
]}
```

If normalization requires multiple steps, extract it as a named tool in `ollie/t`.

## Timeout

`timeout` applies per stage (default: 30s). A pipeline of N stages can run up to N × timeout seconds total. Set explicitly for long-running stages.

## When to use which pattern

| Situation | Pattern |
|---|---|
| One independent operation | single stage |
| N independent operations, same output schema | `{parallel: [...]}` stage |
| N operations feeding one transform | parallel first stage, then sequential |
| Multi-step data transform | sequential stages |
| Reusable transform logic | `{tool}` stage referencing a named script in `ollie/t` |
