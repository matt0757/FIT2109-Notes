# Week 5 Practice Test: Solutions and Learning Notes

## Question 1: Fetch, pull, and push

**Answer: B.**

`git fetch` updates remote-tracking references without changing the working tree or current branch. `git pull` fetches and then integrates the fetched changes into the current branch. `git push` sends local commits to a remote. A local commit never leaves the machine until it is pushed.

**Remember:** commit is local history; push publishes it; fetch inspects remote progress; pull fetches and integrates.

## Question 2: Remote-tracking branches and upstreams

### (a)

`origin/feature-parser` is a remote-tracking branch: a local record of where the remote branch was at the last fetch. It is not the same as an editable local branch.

### (b)

The local `feature-parser` contains two commits that the recorded remote branch does not contain, and it is missing one commit that the remote branch has.

### (c)

Remote-tracking information is only updated by commands such as `git fetch`, `git pull`, or another operation that fetches. “Up to date” without a recent fetch may only mean “up to date with the last known remote state.”

## Question 3: Pull requests and conflicts

### (a)

A pull request is not a Git object or commit stored in the repository. It is a hosting-platform review workflow that compares a source or compare branch with a target or base branch and provides discussion, checks, and a merge interface.

### (b)

A safe sequence is:

```bash
git pull
# resolve conflicts if Git reports them
git add <resolved-files>
git commit              # finish the merge if required
git push
```

Alternatively, use an explicitly chosen rebase workflow if the team's policy permits it. Pulling first brings the teammate's work into the local history before retrying the push.

### (c)

Remove all conflict markers and choose or combine the correct content. Then run `git status`, stage the resolved file with `git add`, verify the result, and complete the merge with a commit or `git merge --continue` as appropriate. The markers themselves must never remain in the final source.

## Question 4: Merge, rebase, and shared history

### (a)

While on `feature`, run:

```bash
git rebase main
```

This replays the feature commits on top of the current `main` tip.

### (b)

A merge preserves the diverged histories and normally creates a merge commit when both sides have moved. A rebase rewrites the feature commits onto a new base, producing a more linear history without a merge commit in that operation.

### (c)

Rebased commits are new commits with different identities. Rewriting commits that others have already fetched can make their histories diverge and force confusing reconciliation. Rebase local, unpushed work; coordinate carefully before rebasing shared history.

## Question 5: Finding and undoing regressions

### (a)

`git blame` identifies the commit that last changed a particular line. `git bisect` is more useful when the regression may involve behavior across several lines or files: it tests commits between a known-good and known-bad point to find the first bad commit.

### (b)

Normally use `git revert` for a bad commit already pushed to shared `main`. It creates a new commit that reverses the change without rewriting the shared history. `git reset` moves the branch and can discard or rewrite commits, which is appropriate only for carefully controlled local history.

### (c)

`git bisect reset` ends bisect mode and returns the repository to the branch and state from before the bisect began.

**Remember:** use history-preserving operations for shared work; use history-rewriting operations only when everyone affected understands the change.
