
# Shell Scripting Cheatsheet
*Condensed from MIT Missing Semester — Command-line Environment*

---

## 1. Arguments (Special Variables)

| Var | Meaning |
|---|---|
| `$0` | Script name |
| `$1`–`$9` | 1st–9th argument |
| `$@` | All arguments (as list) |
| `$#` | Number of arguments |
| `$?` | Exit code of last command |
| `$!` | PID of last backgrounded job |

```bash
# Flags: -a (short), --all (long), grouped: -la == -l -a
# -- stops flag parsing: touch -- -myfile
# - as filename = "read from stdin": grep "x" -
```

## 2. Globs (Shell Expands BEFORE the Program Runs)

| Pattern | Matches |
|---|---|
| `*` | Zero or more of any char |
| `?` | Exactly one char |
| `{a,b,c}` | Expands to each: `folder/{a,b}.py` → `folder/a.py folder/b.py` |
| `*{.py,.sh}` | Combines — all .py and .sh files |

```bash
mv *{.py,.sh} folder/        # move all .py and .sh
cp project/{setup,build}.sh /newpath
# zsh: **/*.py = recursive
```

## 3. Streams & Redirection

Every program: **stdin** (input) → **stdout** (normal output) + **stderr** (errors).

```bash
cmd > file         # stdout → file (overwrite)
cmd >> file        # stdout → file (append)
cmd 2> file        # stderr → file
cmd &> file        # both stdout + stderr → file
cmd < file         # stdin from file
cmd > /dev/null 2>&1   # discard everything

# Pipes: ALL programs in a pipeline start at once (concurrent)
cat file | grep 'err' | uniq -c
```

 **Pipes only carry stdout** — stderr still prints to your terminal. That's why `ls /nonexistent | grep x` still shows the error.

## 4. Variables & Quoting

```bash
foo=bar        # ✅ NO spaces around =
foo = bar      # ❌ runs program "foo" with args = and bar
```

| Quotes | Behavior |
|---|---|
| `'single'` | **Literal** — no expansion, no escapes |
| `"double"` | Expands `$vars`, runs `$(cmd)` |

```bash
foo=bar
echo "$foo"    # → bar
echo '$foo'    # → $foo
```

## 5. Substitution

```bash
# Command substitution — capture output into a variable
files=$(ls)

# Process substitution — treat a command's output as a FILE
diff <(ls src) <(ls docs)
# Use when a program expects a filename, not stdin
```

## 6. Environment Variables

```bash
TZ=Asia/Tokyo date     # one-shot: only for this command
export DEBUG=1         # persists for all child processes
unset DEBUG            # delete
printenv               # show all current env vars
```

Convention: **ALL_CAPS** for env vars, lowercase for local shell vars.

```bash
# Adding to PATH
export PATH="$PATH:/new/path"
```

## 7. Exit Codes & Logic

**0 = success, nonzero = failure.** This is how `if`, `while`, `&&`, `||` all decide.

```bash
cmd && echo "succeeded"   # run if exit code 0
cmd || echo "failed"      # run if exit code nonzero
exit 1                    # return nonzero from a script

grep -q "pat" file && echo found    # -q = quiet, only exit code

# if uses the COMMAND's exit code directly
if grep -q "pattern" file.txt; then
    echo "Found"
fi

# while reads line by line
while read line; do
    echo "$line"
done < file.txt
```

## 8. Signals & Job Control

| Signal | Trigger | Use |
|---|---|---|
| `SIGINT` | Ctrl-C | Interrupt — programs CAN catch/ignore it |
| `SIGQUIT` | Ctrl-\ | Quit (fallback when SIGINT is ignored) |
| `SIGTERM` | `kill <PID>` | Graceful exit request — can be caught |
| `SIGKILL` | `kill -9 <PID>` | **Cannot be caught** — always kills |
| `SIGTSTP` | Ctrl-Z | Pause/suspend process |
| `SIGHUP` | Closing terminal | Kills children — `nohup` blocks this |

```bash
kill -TERM <PID>       # graceful
kill -9 <PID>          # force
kill -0 <PID>          # check exists — sends nothing, just exit code

sleep 1000 &           # start in background
jobs                   # list jobs
fg %1 / bg %1          # bring to foreground / resume in background
nohup cmd &            # survives terminal close
disown                 # detach an already-running job
pgrep -lf name         # find PID by name
```

**In scripts — cleanup on exit:**
```bash
cleanup() { rm -f /tmp/mytemp.*; }
trap cleanup EXIT                  # runs on script exit
trap cleanup SIGINT SIGTERM        # and on Ctrl-C / kill
```

## 9. SSH

```bash
ssh-keygen -a 100 -t ed25519          # generate key pair
ssh-copy-id -i ~/.ssh/id_ed25519 alice@remote

# Run remote commands
ssh user@host 'ls | wc -l'            # quoted = runs on REMOTE
ssh user@host ls | wc -l              # unquoted = ls remote, wc LOCAL
```

**~/.ssh/config** (also read by scp, rsync):
```
Host vm
    User alice
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

Host *.mit.edu
    User alice
```

**File transfer:** `scp local remote:path` / `rsync` (skips identical files, `--partial` resumes).

## 10. tmux

All commands: **Ctrl+b, release, then key.**

| Keys | Action |
|---|---|
| `<C-b> d` | **Detach** (session keeps running) |
| `<C-b> c` | New window |
| `<C-b> N` | Go to window N |
| `<C-b> ,` | Rename window |
| `<C-b> "` | Split horizontal |
| `<C-b> %` | Split vertical |
| `<C-b> z` | Toggle zoom on pane |
| `<C-b> [` | Scrollback (`<space>` select, `<enter>` copy) |

```bash
tmux new -s name    # named session
tmux ls             # list sessions
tmux a -t name      # reattach
```

Detach + reattach > `nohup` — your sessions survive disconnection, essential over SSH.

## 11. Aliases & Useful Tools

```bash
alias ll="ls -lh"
alias gs="git status"
alias mv="mv -i"              # safer defaults
unalias ll                     # remove
\ls                            # bypass alias once
```

⚠️ Aliases **can't take mid-command arguments** — use a function for that.

```bash
# Handy tools
fzf                    # fuzzy finder: ls | fzf, Ctrl-R integration
tldr <cmd>             # example-based man pages
rg                     # better grep        fd  # better find
command-not-found.com  # how to install anything
```

**Dotfiles:** keep them in a git repo, symlink into place with a script → portable, versioned, one-minute setup on any new box.

---

## 🧠 The 5 Gotchas That Bite Everyone

1. **`foo = bar` fails** — no spaces in assignments
2. **Single quotes don't expand** — `'$foo'` is literal text
3. **`if` uses exit codes, not booleans** — `if grep -q ...` not `if [ ... ]` for command checks
4. **Pipes skip stderr** — errors bypass the pipeline and still print
5. **Backgrounded jobs die with the terminal** — use `nohup`, `disown`, or tmux

---

Want me to add a "pentest-flavored" version — quick patterns like loops over targets, output capture in scripts, or trap-based cleanup for temp files?