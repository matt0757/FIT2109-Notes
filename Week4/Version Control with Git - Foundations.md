# Week 4: Version Control with Git - Foundations

## 4.1 Version Control & Git
- Version control = tracking file changes over time (see history, compare, revert).
- Without it: messy manual copies (e.g. `file_v2.doc`), no real history.
- History: RCS (single files) → CVS/SVN (central server, team) → BitKeeper/Mercurial (distributed, local repos).
- Git made by Linus Torvalds (2005) after losing free BitKeeper access.
- Git = distributed VCS: every user has **full project history**, not just latest files.
- Three jobs: **history** (see past), **recovery** (undo mistakes), **coordination** (team work).
- **Snapshots, not diffs**: each commit = full snapshot of files at that moment (unchanged files reused, not restored).
- **Distributed**: every clone = full copy of repo, not just partial checkout. Works offline.
- Git = a workflow: stage → commit deliberately, use branches to isolate experiments.

## 4.2 Setting Identity (`git config`)
- Git needs author name/email before committing.
```
git config --global user.name "Name"
git config --global user.email "email"
```
- `--global` = set once per machine (stored in `~/.gitconfig`). Omit `--global` to override per-project.
- Check settings: `git config --get user.name`, or `git config --list`.
- **Line endings**: Windows = CRLF, Linux/Mac = LF. Wrong setting breaks shell scripts.
  - Fix: `git config --global core.autocrlf input` (for WSL/Linux use).
  - Broken script symptom: `env: 'bash\r': No such file` → fix with `dos2unix`.

## 4.3 Git Repository Model
- Three areas: **working tree** (your files), **staging area/index** (next commit prep), **Git directory** (metadata/history storage).
- Three file states: **modified** (changed, not staged), **staged** (marked for next commit), **committed** (saved in history).
- `git status` shows differences between working tree, index, and last commit (HEAD).
- `git add` = "stage exactly this content now," not just "track this file."
- Basic flow:
```
git init
git add file
git status
git commit -m "message"
```
- Set default branch name once: `git config --global init.defaultBranch main`

## 4.4 Commits & History as a DAG
- A **commit** = snapshot + metadata (author, message, parent pointer(s)).
  - 0 parents = initial commit, 1 parent = normal, 2+ parents = merge commit.
- History = **DAG** (Directed Acyclic Graph): pointers go backward, never loop.
- A **branch** = movable label pointing to a commit, not a copy of history.
- `git log` = graph traversal tool, not just a list.

| Command | Use |
|---|---|
| `git log` | Full commit details |
| `git log --oneline` | Short hash + message |
| `git log --oneline --graph --decorate --all` | ASCII view of branching |
| `git log -5` | Last 5 commits |
| `git log --author="name"` | Filter by author |
| `git log A..B` | Commits in B not in A |

- Merge commit ties two diverged lines back together (has 2 parents).

## 4.5 Tags
- A **tag** = pointer to one specific commit that never moves (unlike branches).
- Used to mark releases (e.g. `v1.0`).
- Two types:
  - Lightweight: `git tag v1.0`
  - Annotated (recommended): `git tag -a v1.1 -m "message"` (stores message/date/author).
- View tags: `git tag` (list all), `git tag -l "v1.*"` (filter).
- Inspect a tag: `git show v1.1`
- View project at that tag: `git switch --detach v1.1` → **detached HEAD** (look only, don't commit here).
- Return to normal: `git switch main`
- Tags are **not** sent to remote automatically when pushing (must push separately — Week 5).

## 4.6 Core Git Operations
| Command | Use |
|---|---|
| `git init` | New empty repo |
| `git clone <url>` | Copy existing repo + full history |
| `git status` | What's staged/modified/untracked |
| `git add <file>` | Stage snapshot of file |
| `git commit -m "..."` | Save staged snapshot |
| `git diff` | Unstaged changes |
| `git diff --staged` | Staged changes |
| `git log` | History |

**New project**: `git init` → create file → `git status` (untracked) → `git add` → `git commit`. First commit = "root-commit" (no parent).

**Editing existing file**: change file → `git status` (shows "modified") → `git add` → `git diff --staged` (review) → `git commit`.

**Joining existing project**: `git clone <url>` → `cd` into folder → `git status` (clean, synced) → `git log` (see teammates' past commits).

- Basic cycle: get repo → inspect → change → stage → review → commit → inspect history.

## 4.7 Ignoring Files (`.gitignore`)
- `.gitignore` = list of file/folder patterns Git should skip (build files, secrets, editor junk).
- Example:
```
__pycache__/
*.pyc
node_modules/
.vscode/
```
- `/` = matches folder, `*` = wildcard (within one path part), `#` = comment, `!` = un-ignore a pattern.
- **Catch #1**: only affects untracked files. Already-tracked files must be removed manually:
```
git rm --cached secrets.env
echo "secrets.env" >> .gitignore
git add .gitignore
git commit -m "Stop tracking secrets.env"
```
  - Removing from tracking does NOT erase it from old commits/history.
- **Catch #2**: `.gitignore` itself SHOULD be committed (shared with team). Personal-only ignores go in a global file: `git config --global core.excludesFile ~/.gitignore_global`

## 4.8 Clean Commit History (8 Habits)
1. **Logical grouping** — one commit = one purpose. Unrelated changes = separate commits. Use `git add -p` to stage partial file changes (hunks).
2. **Inspect before commit** — always check `git diff --staged` to catch debug code, mistakes, etc.
3. **Meaningful messages** — short imperative title (e.g. "Add input validation"), blank line, then details if needed. Avoid vague messages like "update".
4. **Avoid `git commit -a`** carelessly — it commits ALL tracked changes, may include things you didn't mean to (e.g. test configs).
5. **Amend, don't stack** — fix the very last commit (message or forgotten file) using:
```
git commit --amend --no-edit      # keep message, add missed file
git commit --amend -m "new msg"   # just fix message
```
   - **Only amend commits not yet shared** with others (amending rewrites history).
6. **Commit at meaningful checkpoints** — not too rare (lose recovery points), not too frequent (noisy history).
7. **Keep every commit working** — project should run/pass tests at each commit (important for `git bisect` later).
8. **Never commit secrets** — API keys/passwords must never enter history; deleting later doesn't remove them from old snapshots. Rotate any leaked secret immediately.

## 4.9 Branching and Merging
- A branch = lightweight, movable pointer to a commit (not a full copy).
- Workflow:
```
git switch -c feature-readme    # create + switch to new branch
# edit, add, commit
git switch main
git merge feature-readme
```
- **Fast-forward merge**: if main hasn't moved, Git just moves the pointer forward (no new commit).
- **True (3-way) merge**: if branches diverged, Git creates a **merge commit** with 2 parents — history becomes non-linear (visible with `git log --graph`).

## 4.10 Conflict Resolution & Branch Cleanup
- **Merge conflict**: happens when same part of a file changed differently on both branches.
- Git marks conflicts with `<<<<<<<`, `=======`, `>>>>>>>` in the file.
- Resolve flow:
```
git merge feature-branch
git status              # shows unmerged paths
# manually edit conflicted file
git add conflicted-file.txt
git commit               # finalizes the merge
```
- Cleanup after merge:

| Command | Use |
|---|---|
| `git branch -d name` | Delete branch (only if fully merged — safe) |
| `git branch -D name` | Force delete (even if unmerged — risky) |
| `git branch` | List local branches |

## 4.11 Saving Work with `git stash`
- Use when you need to switch branches but aren't ready to commit current work.
- `git stash` shelves changes (staged + unstaged) and resets working tree to match HEAD.

| Command | Use |
|---|---|
| `git stash` | Shelve changes, clean working tree |
| `git stash pop` | Reapply latest stash + remove from list |
| `git stash apply` | Reapply latest stash but keep it in list |
| `git stash list` | Show all stashes |
| `git stash drop` | Delete a stash without applying |

- By default only tracked files are stashed. Include untracked files with:
```
git stash push -u -m "WIP: message"
```
- Stash is **local only** — never pushed to remote, not part of commit history. Use for temporary "switch away right now" needs, not as a commit substitute.