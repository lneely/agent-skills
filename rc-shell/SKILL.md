---
name: rc-shell
intent: scripting, shell
description: Plan 9 rc shell scripting syntax and conventions. Use when writing .rc shell scripts or working with Plan 9 rc syntax.
user-invocable: false
---

# Plan 9 rc Shell Syntax

Shebang: `#!/usr/bin/env rc` Extension: `.rc`

## Variables

```
name=value                   # NO spaces around =
files=(a b c)                # lists in parens
$name $files(1) $files(2-)   # substitute, 1st element, from 2nd onward
$files(2-4)                  # range
$#files                      # count
$"files                      # join with spaces
$^files                      # concat (no spaces)
```

## Command Substitution

```
result=`{command}            # backticks, NOT $()
result=`split{command}       # split using 'split' instead of $ifs
```

## Control Flow

```
if(test -f file) {
    echo exists
}

for(i in 1 2 3) { echo $i }
for(i) { echo $i }           # iterates over $* if no list

while(test $n -lt 10) { n=`{echo $n + 1 | bc} }

switch($file) {
case *.txt
    echo text
case *
    echo other
}
```

**`if not` is unreliable** — though documented in the rc manual, `if not` causes syntax errors in practice. Use a flag variable instead:

```
ok=0
if(test -f file) { echo exists; ok=1 }
if(~ $ok 0) echo missing
```

Chained `if` statements serve as logical AND:

```
if(test -f file) if(~ $mode rw) echo writable
```

## Functions & Pattern Matching

```
fn name {                    # always use multi-line bodies
    echo $* $1               # $* all args, $1 first arg
}
fn name                      # remove definition
~ $file *.txt                # pattern match (sets $status)
if(~ $#list 0) { echo empty }
```

**Single-line function bodies with `;` cause syntax errors.** Always use multi-line:

```
# WRONG: fn die { echo $* >[1=2]; exit 1 }
fn die {
    echo $* >[1=2]
    exit 1
}
```

## Redirection & Pipes

```
>out >>out <in <<EOF         # stdout, append, stdin, here doc
>[2]err >[2=]                # fd 2 to file, close fd 2
>[1=2]                       # stdout to stderr (write to stderr)
>[2=1]                       # stderr to stdout
<[0=3]                       # fd 0 from fd 3
<{cmd} >{cmd} <>{cmd}        # process substitution
cmd1 | cmd2                  # pipe
|[2] |[1=2]                  # pipe fd 2, pipe fd 2 to fd 1
{ cmd1; cmd2 } > file        # redirect block stdout to file
```

## Operators

`; & && || ! @` - sequence, async, and, or, invert, subshell
`^` - concatenate (auto-inserted without whitespace)

## Special Variables

`$*` `$status` `$apid` `$pid` `$ifs` `$path` `$home`

## Built-ins

`. file` `cd` `eval` `exec` `exit` `shift` `wait` `whatis`

## Key Differences from Bash

- Backticks `{cmd}`, not `$(cmd)`
- Lists `(a b c)`, 1-indexed
- No reliable `if/else`: use flag variable + chained `if` instead of `if not`
- `test` or `~`, not `[[ ]]`
- `$#var` not `${#var}`, `$"var` not `"$*"`
- `>[1=2]` to write to stderr (stdout→stderr), `>[2=1]` for stderr→stdout
- `>[2]file` redirects fd 2 to a file; bare `>[2]` (no filename) is a syntax error
- No `${var:-default}`
- `fn name` not `function name`
- Single quotes only (double quotes not special)
- Free carets: `$x.c` → `$x^.c`
