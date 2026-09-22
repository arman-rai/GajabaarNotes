# Vim & Dev Environment Cheatsheet (AI)
*Condensed from MIT Missing Semester — Development Environment and Tools*

---

##  The One Idea

**Vim's interface is a programming language.** Keystrokes are composable commands:

```
[count] [verb] [noun]
   3        d        w      = "delete 3 words"
            c        i(     = "change inside the parens"
```

You don't memorize 200 hotkeys — you learn ~15 verbs and ~15 nouns, and the grammar generates everything else. That's why it feels like a brain-computer interface once it's muscle memory.

---

## Survival Kit (stuck in Vim right now?)

```
i        start typing (Insert mode)
<ESC>    back to Normal mode
:wq      save and quit
:q!      quit WITHOUT saving (your panic exit)
```

---

## 1. Modes

| Mode | Enter with | Purpose |
|---|---|---|
| **Normal** *(default)* | `<ESC>` from anything | Navigate + edit — you live here |
| Insert | `i` | Type text like a normal editor |
| Replace | `R` | Overwrite text |
| Visual | `v` | Select characters |
| Visual Line | `V` | Select whole lines |
| Visual Block | `<C-v>` | Select a column block |
| Command-line | `:` | Run commands (`:w`, `:q`, `:42`) |

⚠️ **Same key, different meaning per mode** — `x` types "x" in Insert, deletes char under cursor in Normal, deletes selection in Visual.

💡 **Remap Caps Lock → ESC.** You press ESC more than any other key.

---

## 2. Movement = Nouns

| Category | Keys | Meaning |
|---|---|---|
| Basic | `h j k l` | left, down, up, right |
| Words | `w` / `b` / `e` | next word / beginning / end of word |
| Line | `0` / `^` / `$` | start of line / first non-blank / end of line |
| Screen | `H` / `M` / `L` | top / middle / bottom of screen |
| Scroll | `<C-u>` / `<C-d>` | half page up / down |
| File | `gg` / `G` | start / end of file |
| Go to line | `:42<CR>` or `42G` | jump to line 42 |
| Matching | `%` | jump to matching bracket/paren |
| Find in line | `f{x}` / `t{x}` | jump **to** / just before next `x` on line |
| | `F{x}` / `T{x}` | same, backwards |
| | `;` / `,` | repeat find forward / backward |
| Search | `/regex` then `n` / `N` | search file, next / previous match |

---

## 3. Edits = Verbs

| Key | Action | Note |
|---|---|---|
| `i` | enter Insert mode | |
| `o` / `O` | open new line below / **above** | exits into Insert mode |
| `d{motion}` | delete motion | `dw` `d$` `d0` — also *copies* what it deletes |
| `c{motion}` | change motion | **delete + enter Insert** — `cw` |
| `x` | delete char | = `dl` |
| `s` | substitute char | = `cl` |
| `u` / `<C-r>` | undo / redo | |
| `y` | yank (copy) | |
| `p` | paste | |
| `~` | flip case of char | |
| `J` | join lines | |

**Selection + verb:** enter Visual (`v`/`V`/`<C-v>`), move to select, then `d`, `c`, or `y` to act on it.

---

## 4. Counts

Prefix anything with a number:

```
3w      move 3 words forward
5j      move 5 lines down
7dw     delete 7 words
```

💡 In practice, for small counts people just mash the key (`ww` instead of `2w`) — use counts when it's genuinely faster.

---

## 5. Modifiers: `i` = inner, `a` = around

The killer feature. Target text *inside* or *around* delimiters:

```
ci"     change inside quotes         "foo" → "|"
ci(     change inside parentheses    (foo) → (|)
ci[     change inside brackets       [foo] → [|]
da'     delete around single quotes 'foo' → |
```

Read them out loud: "**c**hange **i**nside **(**".

---

## 6. Worked Example — Fixing Fizz Buzz

The whole point in one sequence. Broken code:

```python
def fizz_buzz(limit):
    for i in range(limit):        # should be range(1, limit + 1)
        ...
        print("fizz", end="")    # line 6 should say "buzz"

def main():
    fizz_buzz(20)
                                  # main() is never called
```

**Fix 1 — main never called:**
```
G                       jump to end of file
o                       open line below, Insert mode
if __name__ == "__main__": main()
<ESC>
```

**Fix 2 — starts at 0:**
```
/range<CR>              search for "range"
ww                      forward 2 words
i  1,  <ESC>            insert "1," → range(1, limit)
e                       jump to end of next word
a  + 1  <ESC>           append → limit + 1
```

**Fix 3 — wrong string on line 6:**
```
:6<CR>                  go to line 6
ci"                     change inside the quotes
buzz<ESC>
```

Notice: no arrow keys, no mouse, and each fix is a short *sentence* of commands.

---

## 7. Common Combos Worth Burning Into Muscle Memory

| Combo | Meaning |
|---|---|
| `dd` | delete (cut) line |
| `yy` | yank line |
| `cc` | change line |
| `D` / `C` | delete / change to end of line (= `d$` / `c$`) |
| `diw` | delete inner word |
| `yaw` | yank a word (with trailing space) |
| `A` | append at end of line (= `$a`) |
| `.` | **repeat last edit** — the most underrated key |
| `:s/old/new/g` | substitute on current line |

---

## 8. Vim Mode Everywhere

You don't have to use Vim itself — enable Vim mode in everything and build reflexes everywhere:

- **VS Code** → VSCodeVim plugin
- **Zsh** → built-in Vim emulation
- Even AI coding tools have Vim editor modes

---

## 9. The Rest of the Lecture (quick hits)

**Language servers (LSP)** — what gives IDEs their brains: code completion, hover docs, jump-to-definition, find references, import management, live linting/formatting. Examples: Pylance (Python), gopls (Go). Point them at your environment (e.g. your venv) or they can't see your packages.

**AI dev, 3 form factors:**
1. **Autocomplete** — steers best via *comments* (`# extract all Markdown links...`) and good function names
2. **Inline chat** — select code, prompt an edit ("use built-in libraries instead")
3. **Coding agents** — multi-step tasks

**IDE extensions worth knowing:** dev containers (isolated tooling), Remote SSH (develop on beefy remote machines), Live Share (Google-Docs-style collaborative editing).

---

## 10. Learning Path

```
vimtutor        # interactive tutorial, comes WITH vim — do this first
VimGolf         # solve challenges in fewest keystrokes
Vim Adventures  # learn via a game
Practical Vim   # the book, once you're hooked
```

**The method:** learn the fundamentals above → enable Vim mode in ALL your software → use it daily. When something feels inefficient, Google it — there *is* a better way.

---
