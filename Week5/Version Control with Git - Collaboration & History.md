# Week 5: Version Control with Git - Collaboration & History

## 5.1 Remotes, Upstream Tracking, Synchronising
- **Remote** = named link to another copy of the repo (usually on a server/platform like GitHub).
- `git remote add <name> <url>` — save that link locally.
- Default remote name from `git clone` = **origin**.
- **Remote-tracking branch** (e.g. `origin/main`) = local record of where the remote's branch was, last time you checked. NOT an editable branch — just a snapshot/note.
  - "Up to date with origin/main" = compared to your last fetch, not a live check.

### fetch / pull / push

| Command | Direction | Updates files? | Updates your branch? | Updates remote? |
|---|---|---|---|---|
| `git fetch` | remote → you | No | No | No |
| `git pull` | remote → you | Yes | Yes | No |
| `git push` | you → remote | No | No | Yes |

- `git fetch` = only updates remote-tracking branches (safe, doesn't touch your work).
- `git pull` = fetch + merge into your current branch.
- `git push` = uploads your local commits (e.g. `git push origin main`).
- Pull does nothing if there's nothing new to fetch.

### Upstream tracking
- A local branch can be linked to a remote branch = its **upstream** (default partner for pull/push).
- Clone auto-sets upstream for the main branch.
- New branches have no upstream until first push with `-u`: `git push -u origin <branch>`.
- Check config:
```
git remote -v        # list remotes + URLs
git branch -vv        # branches + upstream + ahead/behind count
```
- "Ahead/behind" count is only as fresh as your last fetch — fetch before judging branch state.

## 5.2 Collaborative Git Workflows

### Basic remote workflow (small team, both on main)
```
git clone <repo-url>
cd <repo-name>
# edit, stage, commit locally (same as Week 4)
git push                # publish commits
```
- Committing = local only. Git never auto-sends anything — push is a separate, deliberate step.
- `git push` needs no args here because clone already set upstream.

### Pull–work–push loop
```
git pull                # bring in teammates' work
# edit, stage, commit
git push                # send yours up
```
- Pull first = cheaper than fixing conflicts after building on outdated code.
- Push only sends **committed** work — uncommitted/staged-but-not-committed changes go nowhere.
- If teammate pushed first, your push is **refused** (not an error — Git protecting their work). Fix: pull, resolve, then push.

### Publishing your own branch (real team workflow)
- Don't commit straight to main — instead: branch → commit → push branch → review → merge.
```
git switch main
git pull                          # update main first
git switch -c feature-parser    # branch from updated main
# edit, add, commit
git push -u origin feature-parser   # first push needs -u (sets upstream)
```
- After first push with `-u`, later `git push`/`git pull` on that branch works without extra args.

### Pull Requests (PR)
- **Not a Git object** — it's a feature of the hosting platform (GitHub/GitLab), not stored in the repo itself.
- GitHub = "pull request", GitLab = "merge request" (same idea).
- Compares: **base branch** (target, usually main) vs **compare branch** (your feature branch).
- PR = review layer on top of Git's branch/commit model.
- When opening one, you specify: base branch, compare branch, title/description, optional reviewers.
- Branch = technical unit of work. PR = social/review unit around that branch.
- Keep branches focused, commits meaningful, PRs small (easier to review).

### Full PR cycle
```
git switch main
git pull
git switch -c feature-readme
# edit, add, commit
git push -u origin feature-readme
# open PR on platform
```
- `git pull` already includes a fetch — no need to fetch separately before it.
- Review = iterative: reviewer comments → you add commits → PR auto-updates.

### Keeping your branch current during review
- The longer a branch goes without main's new commits, the more likely conflicts appear.
- Fix: merge main into your branch periodically (not just once at the end):
```
git switch main
git pull
git switch feature-readme
git merge main        # resolve conflicts now, while small
git push
```
- Alternative (no merge commit): **rebase** (see 5.4).

## 5.3 Merging & Conflict Resolution (Revisited)
- Most merges are automatic and silent (different files or different lines = no conflict).
- **Conflict** = same region of same file changed differently on both sides — Git can't choose, so it stops and asks you.
- Same happens in PRs: platform disables "Merge" button until conflict resolved.
- Prevention: sync often (small conflicts are easier than big ones).

### How a conflict happens (example)
- Teammate changes a line, commits, pushes. You (unaware) change the same line differently, commit.
- Your push is refused (remote has commits you don't) → you pull → conflict appears:
```
git pull
# CONFLICT (content): Merge conflict in greeting.py
```
- File shows both versions:
```
<<<<<<< HEAD
message = "Welcome"
=======
message = "Hello, world!"
>>>>>>> 9d4e7b0 (Use a friendlier greeting)
```
- Above `=======` = your version (HEAD). Below = incoming version (labelled with branch/commit).
- Resolving = your decision — pick one side, combine both, or write something new.

### Resolve and finish
```
git status                  # see which files are conflicted
# edit file(s), remove all <<<<<<< ======= >>>>>>> markers
git add <file>               # marks as resolved (no separate command)
git status                  # confirm nothing left unmerged
git merge --continue        # records the merge commit
```
- Result = normal merge commit with 2 parents (yours + teammate's).

### Abort instead
```
git merge --abort
```
- Cancels the in-progress merge, restores branch/working tree to before the merge started. Your & teammate's commits are untouched — only the in-progress merge disappears.
- Caveat: uncommitted changes before the merge started may not be recoverable.

## 5.4 Rebasing
- **Rebase** = second way (besides merge) to combine branches — it moves your commits onto a new base, instead of joining two histories with a new commit.
```
git rebase <base>
```
- The branch that moves = the one you're currently on. `<base>` stays untouched.
- `<base>` can be a branch, remote-tracking branch, tag, or commit hash.
- **Rule**: `git switch <branch to move>` then `git rebase <branch to put it on top of>`.
```
git switch feature      # branch to move
git rebase main        # new base
```
- Result: feature branch looks like it was created from latest main — linear history (no merge commit).

### Merge vs rebase
- Merge = preserves that branches diverged and were combined (extra merge commit).
- Rebase = rewrites your branch's base, replaying commits — linear history, no merge commit.

### Two common uses
1. **Update feature branch before merging (avoid merge commit):**
```
git switch main
git pull
git switch feature
git rebase main
```
   Or skip main entirely:
```
git fetch origin
git rebase origin/main
```
2. **Pulling without a merge commit:**
```
git pull --rebase
```
   Make it default everywhere:
```
git config --global pull.rebase true
```

### Rebase rewrites history
- Rebased commits are NEW commits (different identity — parent changed).
- Safe for **local, unpushed** work. Dangerous for **already-shared/pushed** commits (confuses collaborators).

### Resolving conflicts during rebase
- Rebase re-applies commits **one at a time** — can pause at each conflicting commit (0 to N times for N commits).
```
git rebase main
# stops at first conflicting commit
git status                    # shows conflicted files + which commit
# edit files, remove markers
git add <file>
git rebase --continue        # repeat if it pauses again
```
- **Important**: do NOT use `git commit` to finish a rebase step — always `git rebase --continue`. (`git commit` creates a stray extra commit and leaves rebase unfinished.)

### Abort a rebase
```
git rebase --abort
```
- Returns branch to exactly where it was before rebase started (original commits/identities). Any resolution done so far is discarded.

## 5.5 Reading & Interpreting History
| Command | Answers |
|---|---|
| `git log` | Which commits are reachable from here? |
| `git log A..B` | Commits in B but not A |
| `git log --follow -- file` | File's history, even across renames |
| `git diff` | What's changed but not staged |
| `git diff --staged` | What will be committed next |
| `git diff A B` | Difference between two commits |
| `git blame file` | Who/what commit last changed each line |
| `git show <commit>` | What one commit changed + its message |

- `--` separates commit/branch names from file paths (avoids ambiguity, e.g. `git log -- README.md`).
- `HEAD~N` = N commits back from current (`HEAD~1` = previous commit, etc).
- `git diff HEAD~1 HEAD` = what changed in the most recent commit.

### Three common real questions
1. **What have teammates done since I last looked?**
```
git fetch origin
git log main..origin/main --oneline
```
2. **What's on my branch that main doesn't have?** (run before opening a PR)
```
git log main..feature-parser --oneline
```
3. **Where did this line come from?** (2-step "code archaeology")
```
git blame parser.py        # find the commit
git show <hash>            # see what/why it changed
```

- Good commit messages → useful `git log`. Focused diffs → useful `git diff`. Small commits → useful `git blame`.

## 5.6 Tracing Regressions with `git bisect`
- **Regression** = something that used to work, now doesn't. Hard to trace when many commits sit between "worked" and "broken."
- `git bisect` = binary search through commit history to find the exact bad commit.
- Efficient: ~10 tests for 1,000 commits (log scale), instead of checking one by one.

### Worked example
```
git bisect start
git bisect bad              # current commit is broken
git bisect good v1.0         # this old commit/tag was fine
```
- Git checks out the midpoint commit for you to test:
```
python -m pytest tests/test_login.py
git bisect good   # or: git bisect bad
```
- Repeat until Git names the first bad commit.

### Must-know cautions
- Use the **exact same test** every time — one wrong verdict ruins the whole search.
- During bisect you're in **detached HEAD** (like tags in Week 4) — no branch moves, nothing of yours is disturbed.
- Bisect isn't done just because the culprit is named — you're still in bisect mode.

### After finding the bad commit
```
git show <bad-commit>          # read what it did (doesn't move anything)
git bisect reset               # end bisect, return to original branch
git switch -c fix/login-regression   # branch off current code (not the old bad commit)
# edit, add, commit fix
```
- If the bad commit is already shared, safer to add a NEW fix commit rather than editing history.
- Bisect works best with small, focused commits (easier to diagnose one commit's effect).

### bisect vs blame
- `blame` = which commit last changed this **line**.
- `bisect` = which commit introduced this **behavior change** (works across multiple files/lines, based on actual testing).

## 5.7 Recovering from Mistakes
- Two main undo tools: `git revert` and `git reset` — solve different problems.

### Which to use?
- **Has the commit been pushed / could others have it?** → use `git revert` (adds a new commit, never rewrites shared history).
- **Still only on your machine?** → `git reset` is fine (can remove the commit entirely).

### `git revert` (safe for shared/pushed history)
- Creates a NEW commit that undoes an earlier one — original commit stays in history.
```
git revert <bad-commit>
git push
```
- Safe because it only adds commits (push works normally, nobody's history is broken).
- Can even "revert the revert" later if needed.

### `git reset` (for local-only cleanup)
```
git reset --soft HEAD~1     # undo commit, keep everything staged
git reset --mixed HEAD~1    # undo commit + unstage (default mode)
git reset --hard HEAD~1     # undo commit + discard all changes (DESTRUCTIVE)
```

| Mode | Staging area | Working tree | Result |
|---|---|---|---|
| `--soft` | untouched | untouched | changes stay staged, ready to recommit |
| `--mixed` (default) | reset | untouched | changes become unstaged edits |
| `--hard` | reset | reset | changes discarded completely |

- **`--soft`** useful for splitting one messy commit into two clean ones:
```
git reset --soft HEAD~1
git restore --staged utils.py
git commit -m "Fix email regex"
git add utils.py
git commit -m "Reformat utils.py"
```
- **`--hard`** is the only destructive one — permanently loses uncommitted work. Old commit itself still exists in repo (just unreferenced) until Git cleans it up, but any *uncommitted* edit is gone forever. Always run `git status` first; treat `--hard` as irreversible unless certain.

### The rule that ties it together
- Local, unshared history = flexible (reset is fine).
- Shared/pushed history = needs care (use revert, not reset — don't rewrite what others already have).