---
name: edit-text
intent: read, write, edit, search, grep, glob, find, cat, sed, awk, file
description: Guidance on using file_* tools for all file read/write/edit/search operations instead of shell commands.
---

# File Editing

Use the `file_*` built-in tools for all file operations. Do not use shell commands (`cat`, `sed`, `awk`, `grep`, `find`, `echo >`, etc.) for file I/O — the file tools are faster, safer, and produce better diffs.

## Tools

| Tool | Use for |
|------|---------|
| `file_read` | Reading a file or a line range |
| `file_write` | Writing a new file or full overwrite |
| `file_edit` | Precise string replacement in an existing file |
| `file_glob` | Finding files by name pattern |
| `file_grep` | Searching file contents by regex |

## Rules

- Read before writing. Always call `file_read` on a file before `file_write` or `file_edit`.
- Prefer `file_edit` over `file_write` for partial changes — it sends only the diff.
- Use `file_glob` instead of `find` or `ls` for file discovery.
- Use `file_grep` instead of `grep` or `rg` for content search.
- Reserve `execute_code` for operations that genuinely require a shell (build, test, git, process control).

## Examples

```
file_read: {"path": "src/main.go"}
file_edit: {"path": "src/main.go", "old": "foo()", "new": "bar()"}
file_glob: {"pattern": "src/**/*.go"}
file_grep: {"pattern": "func.*Handler", "path": "src/"}
```
