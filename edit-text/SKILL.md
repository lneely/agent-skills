---
name: edit-text
intent: read, write, edit, search, grep, glob, find, cat, sed, awk, file
description: Guidance on using file_* tools for all file read/write/edit/search operations instead of shell commands.
---

# File Editing

**Always use `file_*` tools for file operations. Never use shell commands for file I/O.**

Using `cat`, `sed`, `awk`, `grep`, `find`, `echo >`, or similar shell commands for reading, writing, or searching files is **wrong**. The `file_*` tools exist precisely to replace them. Use them every time, without exception.

## Tools

| Tool | Replaces |
|------|---------|
| `file_read` | `cat`, `head`, `tail`, `less` |
| `file_write` | `echo >`, `tee`, `cat >` |
| `file_edit` | `sed -i`, `awk`, manual patch |
| `file_glob` | `find`, `ls` |
| `file_grep` | `grep`, `rg` |

## Rules

- **Read before writing.** Always call `file_read` on a file before `file_write` or `file_edit`. This is enforced — writes will fail without a prior read.
- **Prefer `file_edit` over `file_write`** for partial changes.
- **Never use `execute_code` or `execute_tool` for file I/O.** Reserve those for operations that genuinely require a shell: building, testing, git, process control.

## Examples

```
file_read: {"path": "/abs/path/to/file.go"}
file_edit: {"path": "/abs/path/to/file.go", "old": "foo()", "new": "bar()"}
file_glob: {"pattern": "src/**/*.go"}
file_grep: {"pattern": "func.*Handler", "path": "src/"}
```
