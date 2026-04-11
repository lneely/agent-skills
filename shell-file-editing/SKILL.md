---
name: shell-file-editing
intent: file editing, text editing, ed, sed, ssam
description: Recipes for reading and editing files using standard Unix tools (ed, sed) and plan9port ssam. Use when modifying files via execute_code.
user-invocable: false
---

# Shell File Editing

Use `execute_code` with standard tools. Never guess line numbers — always grep first.

## Locate Before Editing

```sh
grep -n 'pattern' file          # line numbers for matches
cat -n file                     # full file with line numbers
grep -n 'pattern' file | head   # narrow to area of interest
```

## ed — Scripted Line Editing

`ed` operates on whole lines. Drive it with a heredoc; use `-s` to suppress noise.

### Address syntax

```
n           line number (1-based)
$           last line
.           current line (last affected)
,           whole file (shorthand for 1,$)
/pattern/   next line matching pattern
?pattern?   previous line matching pattern
n,m         line range inclusive
/pat1/,/pat2/  range from first match to second
```

### Substitute on a matching line

```sh
ed -s file <<'EOF'
/pattern/s/old/new/
w
q
EOF
```

### Global substitute (all occurrences in file)

```sh
ed -s file <<'EOF'
,s/old/new/g
w
q
EOF
```

### Delete lines matching a pattern

```sh
ed -s file <<'EOF'
g/pattern/d
w
q
EOF
```

### Delete a line range

```sh
ed -s file <<'EOF'
5,10d
w
q
EOF
```

### Insert text before a matching line

```sh
ed -s file <<'EOF'
/pattern/i
new line of text
.
w
q
EOF
```

### Append text after a matching line

```sh
ed -s file <<'EOF'
/pattern/a
new line of text
.
w
q
EOF
```

### Replace a range of lines

```sh
ed -s file <<'EOF'
/start_pattern/,/end_pattern/c
replacement line 1
replacement line 2
.
w
q
EOF
```

### Replace exact line range

```sh
ed -s file <<'EOF'
3,7c
new content
.
w
q
EOF
```

### Append at beginning of file (address 0)

```sh
ed -s file <<'EOF'
0a
first line
.
w
q
EOF
```

### Move lines

```sh
ed -s file <<'EOF'
5,8m12
w
q
EOF
```

### Print without modifying (verify before write)

```sh
ed -s file <<'EOF'
,p
q
EOF
```

### Multi-step script

```sh
ed -s file <<'EOF'
/^func foo/,/^}/c
func foo(x int) int {
	return x * 2
}
.
,s/OldName/NewName/g
w
q
EOF
```

### Key points

- Input mode (`a`, `i`, `c`) ends with a lone `.` on its own line
- `g/re/cmd` runs cmd on every matching line; useful with `d`, `s`, `p`
- `wq` writes and quits in one command
- Substitute delimiter can be any non-space non-newline character: `s|/path/a|/path/b|`
- `\n` in replacement introduces a newline (splits the line)

---

## sed — Stream Substitution

Best for simple in-place substitutions. Single-pass; cannot insert multi-line
blocks as easily as ed.

### In-place substitution (GNU sed)

```sh
sed -i 's/old/new/g' file
```

### In-place on multiple files

```sh
sed -i 's/old/new/g' *.go
```

### Substitute only on lines matching a pattern

```sh
sed -i '/pattern/s/old/new/' file
```

### Delete lines matching a pattern

```sh
sed -i '/pattern/d' file
```

### Delete a line range

```sh
sed -i '5,10d' file
```

### Print a line range (no modification)

```sh
sed -n '10,20p' file
```

### Multiple expressions

```sh
sed -i -e 's/foo/bar/g' -e 's/baz/qux/g' file
```

### Address forms

```
n           line n
$           last line
/regexp/    lines matching regexp
n,m         range
/a/,/b/     range between patterns
n,+k        line n and k lines after it
```

### Key points

- GNU sed: `-i` with no suffix. BSD sed: `-i ''`. Prefer `ed` for portability.
- sed makes one pass; edits to earlier lines do not affect later address matching
- Use `ed` when inserting or replacing blocks of multiple lines

---

## ssam — Structural Editing (plan9port)

`ssam` uses sam's command language. Selects structurally (by regex), not by
line number. Entire input is loaded before the script runs.

Check availability: `command -v ssam`

### Basic syntax

```
ssam 'address command' file
ssam -n 'address command' file    # suppress default output
```

On invocation, the entire file is selected (dot = `,`).

### Address syntax

```
,           whole file
n           line n
/regexp/    next match of regexp
?regexp?    previous match
#n          character offset n (#0 = start of file)
$           end of file
.           current selection (dot)
a1,a2       from start of a1 to end of a2
```

### Print first 10 lines

```sh
ssam -n ',10p' file
```

### Global substitution

```sh
ssam 's/old/new/g' file
```

### Substitute within lines matching a pattern

```sh
ssam 'x/.*pattern.*\n/ s/old/new/' file
```

### Delete empty lines

```sh
ssam 's/\n\n+/\n/g' file
```

### Print one word per line

```sh
ssam -n 'y/[a-zA-Z]+/ c/\n/' file
```

### Structural loop: run command on each match

```sh
# x selects each match of the pattern, then runs the sub-command on it
ssam 'x/pattern/ c/replacement/' file
```

### Conditional: run command only if range contains match

```sh
ssam 'g/pattern/ s/old/new/' file    # only lines containing pattern
ssam 'v/pattern/ s/old/new/' file    # only lines NOT containing pattern
```

### Key points

- ssam consumes all of stdin before running; safe for in-place style pipelines
- `x/re/ cmd` iterates over all non-overlapping matches; nested `x` is legal
- `y/re/ cmd` iterates over the text *between* matches
- `g/re/ cmd` and `v/re/ cmd` are guards (if-contains / if-not-contains)
- To write back in place: `ssam 'script' file > tmp && mv tmp file`

---

## Workflow

1. Locate with `grep -n` or `cat -n`
2. Preview the change with `ed -s file <<'EOF'\n,p\nq\nEOF` or `sed -n`
3. Apply the edit
4. Verify with `grep` or `cat -n` that the change landed correctly
