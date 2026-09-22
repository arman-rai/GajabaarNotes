	 https://missing.csail.mit.edu/2026/course-shell/
# Notes to myself:
- Try to think about unknown unknowns

# Shell Fundamentals Cheatsheet (AI)
*Condensed from MIT Missing Semester — Course Overview + Introduction to the Shell*

---

## 1. Anatomy of the Prompt

```
missing:~$
  │     │  └─ $ = you (not root)
  │     └──── current working directory (~ = home)
  └────────── machine name
```

```bash
date              # run a program
echo hello        # run program with argument
echo "My Photos"  # quote args with spaces (or escape: My\ Photos)
```

## 2. Navigation

| Command | Action |
|---|---|
| `cd path` | change directory (**shell builtin**, not a program) |
| `pwd` | print working directory |
| `~` | home |
| `.` / `..` | this dir / parent dir |
| `/bin/...` | absolute path (starts with `/`) |
| `bin/...` | relative path (resolved from cwd) |

## 3. How the Shell Finds Programs: `$PATH`

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

which echo        # → /bin/echo  (what actually runs)
/bin/echo hi      # bypass $PATH, run directly
```

## 4. Core Text Tools

| Command | Action |
|---|---|
| `cat file` | print file |
| `sort file` | sort lines |
| `uniq file` | **dedupe consecutive** duplicate lines (→ `sort \| uniq`) |
| `head file` / `tail file` | first / last few lines (`tail -n 20`) |
| `grep pattern file` | lines matching regex (`-r` = recursive, `-i` = case-insens) |

## 5. The Three Power Tools

### sed — programmatic editor
```bash
sed -i 's/pattern/replacement/g' file
#   │   │ │         │            └─ g = all occurrences per line (not just first)
#   │   │ │         └─ replacement
#   │   │ └─ pattern (regex)
#   │   └─ s = substitute command
#   └─ -i = edit file in place
```

### find — search files
```bash
find ~/Downloads -type f -name "*.zip" -mtime +30   # zips older than 30 days
find ~ -type f -size +100M -exec ls -lh {} \;       # files >100M, list them
find . -name "*.py" -exec grep -l "TODO" {} \;      # .py files containing TODO

# -exec runs a command per match: {} = filename, \; = command terminator
```

### awk — column extraction
```bash
awk '{print $2}' file          # 2nd whitespace-separated column of each line
awk -F, '{print $2}' file      # 2nd COMMA-separated column (CSV)
awk '$3 ~ /pattern/ {print}'   # only rows where col 3 matches
```

## 6. Pipes & Redirection

```bash
cmd | next            # stdout of cmd → stdin of next
cmd > file            # stdout → file (overwrite)
cmd >> file           # stdout → file (append)
cmd < file            # stdin from file
cmd | tee log.txt     # show on screen AND save to file (split the stream)
```

Pipes work because most programs read **stdin** when given no file argument.

## 7. Bash as a Programming Language

### Conditionals
```bash
if command1; then command2; command3; fi

[ -f file ]         # test: does file exist? ("test" abbreviated as "[")
[ "$var" = "str" ]  # string equality
[[ -f file ]]        # bash built-in — safer quoting behavior, prefer this
```

### Loops
```bash
while cmd1; do cmd2; done          # repeat while cmd1 succeeds (exit 0)
for f in a b c; do echo $f; done   # iterate over list

for i in $(seq 1 10); do           # command substitution inside loop
    echo $i
done
```

### Command substitution
```bash
files=$(ls)          # capture output of command into variable
# prefer $() over backticks `ls` — $() nests
```

## 8. Shell Script Skeleton (memorize this)

```bash
#!/bin/bash           # shebang: which interpreter runs this file
set -euo pipefail     # STRICT MODE — always use this
#   -e            exit on any command failing
#   -u            crash on undefined variables (instead of empty string)
#   -o pipefail   pipeline fails if ANY part of it fails

stress --cpu 8 &      # & = run in background
STRESS_PID=$!         # $! = PID of last backgrounded job

LOGFILE="test_$(date +%s).log"    # timestamped filename

RUN=1
while cargo test > "$LOGFILE" 2>&1; do     # capture stdout+stderr to file
    ((RUN++))
done

kill $STRESS_PID       # cleanup
echo "Failed on run $RUN"
tail -n 20 "$LOGFILE"  # show the tail of the failed output
```

**Shebang works for any language:** `#!/usr/bin/env python3` at the top of a `.py` file makes it directly executable.

## 9. Dissecting the Famous Pipeline

```bash
ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \   # fetch logs remotely
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \   # extract username via regex capture group → \1
  | sort | uniq -c \            # count occurrences per username
  | sort -nk1,1 | tail -n10 \   # sort by count (col 1, numeric), top 10
  | awk '{print $2}' \          # drop the count, keep just names
  | paste -sd,                  # join lines with commas
```

**Top 10 users who disconnected from SSH, comma-separated.** One command. This is the whole philosophy: small tools, composed.

## 10. Modern Upgrades (from the lecture's footnotes)

| Old | New | Why |
|---|---|---|
| `cat` | `bat` | syntax highlighting + paging |
| `ls` | `eza` | human-friendly listing |
| `find` | `fd` | simpler syntax, saner defaults |
| `grep` | `rg` (ripgrep) | way faster, recursive by default |
| `cd` | `z` (zoxide) | remembers frequent dirs, jumps there |
| `man` | `tldr` | example-first docs |

Plus: **shellcheck** — lints your shell scripts, catches the classic footguns. Use it on anything over ~20 lines. LLMs are also great at writing/debugging bash.

## 11. Gotchas That Bite Beginners

1. **`uniq` only dedupes *consecutive* lines** — always `sort | uniq`, never `uniq` alone
2. **`cd` must be a builtin** — a child process can't change its parent's cwd, so a standalone `cd` binary could never work
3. **`./script.sh` fails without `chmod +x`** — execute permission is separate from read
4. **`foo = bar` is wrong** — spaces split arguments; assignments take no spaces
5. **Script silently "works" with undefined vars** — `$TYPO` just becomes empty string... unless `set -u` is on
6. **`find -exec` needs `\;`** — the semicolon terminates the command; escape it or the shell eats it first

## 12. Exit Codes & Chaining

```bash
$?         # exit status of last command (0 = success)
cmd1 && cmd2   # run cmd2 only if cmd1 SUCCEEDED
cmd1 || cmd2   # run cmd2 only if cmd1 FAILED
```

---

## 🧭 Mental Model

```
Terminal (GUI) → Shell (bash) → runs Programs (via $PATH)
                       │
                       └── is itself a language: | > < $() if/for/while
                                │
                                └── philosophy: small sharp tools, composed via pipes
```

---

That completes the set for the lectures you've sent — you now have: **shell fundamentals → scripting/command-line env → Vim → port forwarding/pivoting**. 

If you keep going through the course, the natural next ones for your CPTS workflow would be the **Data Wrangling** lecture (grep/sed/awk/regex at full power) and **Debugging/Profiling** — say the word and paste the content.