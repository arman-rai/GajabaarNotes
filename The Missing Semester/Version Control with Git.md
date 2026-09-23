# Git Cheatsheet (AI)
*Condensed from MIT Missing Semester — Version Control*

---

##  The One Idea

**Git's interface is ugly, but its design is beautiful. Learn the data model first, and the commands stop being magic incantations.**

The entire lecture in one sentence: *Git stores snapshots (commits) in a DAG, addressed by content hashes, pointed to by mutable human-readable names (references) — and every command is just a manipulation of that graph.*

---

## 1. The Data Model (memorize this pseudocode)

```
blob  = array<byte>                    // a file — just bytes, no name
tree  = map<string, tree | blob>       // a directory — names → blobs/trees
commit = struct {
    parents: array<commit>             // ← DAG, not a list! (merges = 2+ parents)
    author, message
    snapshot: tree                     // top-level tree
}
```

### Content addressing
Every object is stored under its **SHA-1 hash**:

```
objects = map<string, object>          // id = sha1(object)
```

Same content anywhere in history → same hash → stored once. Objects **reference each other by hash**, never embed.

### Verify it yourself
```bash
git cat-file -p <hash>        # print any raw object
# a tree looks like:
# 100644 blob 4448adbf...  baz.txt
# 040000 tree c68d233a...  foo
```

---

## 2. History Is a DAG

```
o <-- o <-- o <-- o <---- o    ← merge commit (2 parents)
            ^             /
             \           v
              --- o <-- o
```

- Arrows point to **parents** ("comes before")
- Branches = just history forking
- Merges = a commit with **multiple parents**

### Immutability
Commits **never change**. "Editing" history (amend, rebase) = creating *new* commits and repointing references at them. The old ones just get orphaned.

---

## 3. References & HEAD

```
references = map<string, commit-hash>   // MUTABLE, unlike objects
```

| Concept | What it is |
|---|---|
| `master` / `main` | just a name pointing at a commit hash |
| `HEAD` | special ref = "where am I now" — what the next commit's parent will be |
| Branch | a movable pointer; committing on a branch just slides it forward |

**Everything Git is:** `objects + references`. All commands manipulate these.

---

## 4. The Staging Area

Git's snapshot ≠ "current state of the folder" (unlike some VCSs). Instead:

```
working dir ──git add──▶ staging area ──git commit──▶ history
```

**Why:** you've written two features + debug prints all at once, but want three *clean* commits. You stage and commit selectively.

---

## 5. Basic Workflow

```bash
git init                          # create repo (.git dir = objects + refs)
git status                        # what's going on
git add <file>                    # stage it
git commit                        # snapshot what's staged
git log --all --graph --decorate  # visualize the DAG
git diff                          # working dir vs staging
git diff <revision>               # vs some snapshot
git checkout <revision>           # move HEAD there
```

**Write good commit messages** — they're the "why" of your history, since the diff only shows "what".

---

## 6. Branching & Merging

```bash
git branch                      # list
git branch <name>               # create
git switch <name>               # switch to it
git checkout -b <name>          # create + switch (older syntax, still common)
git merge <revision>            # merge that branch into current
git mergetool                   # GUI/terminal tool for conflicts
git rebase                      # replay your patches onto a new base
```

### Merge conflict markers
```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> branchname
```
Edit to what you actually want, delete markers, `git add file && git commit` (or `git merge --continue`).

---

## 7. Remotes

```bash
git remote                                              # list
git remote add <name> <url>                             # e.g. origin
git push <remote> <local>:<remote>                       # upload objects + update remote ref
git fetch                                               # download objects/refs (no merge)
git pull                                                # = fetch + merge
git clone <url>                                         # download whole repo
git branch --set-upstream-to=<remote>/<branch>           # link local ↔ remote branch
```

---

## 8. Undo (think: "what graph manipulation do I want?")

```bash
git commit --amend       # replace last commit (new commit, repointed)
git reset <file>         # unstage a file
git restore              # discard working-dir changes
```

**The lecture's trick for anything:** translate your goal into the data model first. *"Discard uncommitted changes and make master point at 5d83f9e"* → `git checkout master; git reset --hard 5d83f9e`. Every goal is a graph edit; there's a command for it.

---

## 9. Advanced (the actually-useful ones)

| Command | Use |
|---|---|
| `git add -p` | **interactive staging** — pick hunks one by one (split debug prints from real fix) |
| `git stash` / `git stash pop` | shelve dirty working dir, come back later |
| `git rebase -i` | interactive rebase: reorder, squash, edit commits |
| `git bisect` | **binary search history** — find which commit broke something |
| `git blame` | who last edited each line (and when) |
| `git revert <sha>` | create a NEW commit that undoes an old one (safe for shared history) |
| `git clone --depth=1` | shallow clone, skip full history |
| `git worktree` | multiple branches checked out simultaneously |
| `git config` | customize everything |
| `.gitignore` | files to never track (creds, build artifacts, .DS_Store) |

Global ignore: `git config --global core.excludesfile ~/.gitignore_global`

Handy alias to set up:
```bash
git config --global alias.graph "log --all --graph --decorate --oneline"
```

---

## 10. Gotchas That Bite People

1. **Git ≠ GitHub.** Git is the tool; GitHub is one host with PR workflows on top. GitLab/BitBucket exist.
2. **`git pull` hides two steps** — fetch + merge. When surprised, `git fetch` first, look, *then* merge.
3. **Amend/rebase rewrite history** — fine on your local commits, dangerous on shared/pushed ones. Use `git revert` instead on public branches.
4. **A branch is just a pointer**, not a copy of files. Creating one is instant and free.
5. **Blob has no filename** — the *tree* holds the name→hash mapping. That's why renaming a file doesn't duplicate its content in the object store.
6. **Committing secrets/large files** — they live in history forever; removing requires history rewriting (filter-repo), not just `rm`.

---

## 11. Resources Worth Bookmarking

- **Pro Git** book, ch. 1–5 — now that you know the model, it'll click
- **Oh Shit, Git!?!** — recovery recipes for common disasters
- **Learn Git Branching** — browser game, best way to build DAG intuition
- **Git for Computer Scientists** — the data model with nice diagrams

---

## 🧭 Mental Model

```
.git/
 ├── objects/          # immutable, content-addressed: blobs, trees, commits (the DAG)
 └── refs/             # mutable: master, HEAD, feature-x...

Every command = add objects and/or move references.
  git commit  → write new commit object, slide current branch pointer
  git merge   → write merge commit (2 parents), move pointer
  git amend   → write NEW commit, repoint (old one orphaned)
```

---
