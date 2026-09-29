# Week 4 Practice Test: Solutions and Learning Notes

## Question 1: Git areas and staging

### (a)

- **Last commit:** the old committed version of `app.py`.
- **Staging area:** the version captured when `git add app.py` was run.
- **Working tree:** the newer version after the second edit.

### (b)

`git add` stages a snapshot of the file's content at that moment; it does not lock the file. A later edit changes the working tree while leaving the earlier snapshot in the index, so the file appears in both staged and unstaged sections.

**Remember:** `git add` means “stage this content now,” not simply “track this filename.”

## Question 2: Commits and branches

**Answer: C.**

A commit records a project snapshot plus metadata and parent pointer(s). A branch is a lightweight movable label pointing to a commit. A merge commit can have two or more parents, while an initial commit has none.

**Common trap:** imagining a branch as a copied folder. Branches are references into the commit graph.

## Question 3: Reading history

### (a)

`0ef44a1`

Both lines lead back to that commit before diverging.

### (b)

It is a **true merge** because `a8f2c11` has two parent lines, joining the `main` and `feature-search` histories. A fast-forward would simply move a branch label without creating a merge commit.

### (c)

`v1.0` is a tag pointing to the specific commit `7d1c008`. Unlike a branch, a tag is intended to remain fixed as a named release point.

## Question 4: Ignoring and recovering files

### (a)

No. `.gitignore` affects untracked files. It does not remove a file that is already in the repository's index.

### (b)

```bash
git rm --cached secret.env
```

This removes it from tracking while retaining the local file. The `.gitignore` entry should then be committed. Old commits may still contain the secret, so the secret must be rotated or revoked.

### (c)

`git stash` temporarily shelves tracked working-tree and staged changes so you can switch context with a clean working tree. It is local and temporary; it is not a commit, not a remote backup, and by default does not include untracked files. Use `git stash push -u` when untracked files must also be included.

## Question 5: Safe commit history

### (a)

`git diff --staged` shows exactly what will enter the next commit. It catches accidental files, debug changes, and incomplete edits before they become history.

### (b)

Amend the most recent commit when it is local and not shared, for example to fix its message or include a forgotten file. Amending rewrites the commit identity and can disrupt collaborators if the commit has already been pushed or based upon.

### (c)

Git history preserves snapshots. Deleting the secret later does not remove it from the earlier commit, and anyone with repository history may still recover it. Never commit secrets; if one leaks, rotate it immediately and clean history as appropriate.

**Remember:** Git is a recovery and coordination system, so deliberate staging and readable history are part of safe collaboration.
