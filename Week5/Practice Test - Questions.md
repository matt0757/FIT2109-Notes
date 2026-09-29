# Week 5 Practice Test: Git Collaboration & History

Attempt the questions before opening the separate solutions file. For collaboration scenarios, distinguish local commits, remote-tracking references, and the hosting platform.

## Question 1: Fetch, pull, and push

Which statement is TRUE?

Select one:

A. `git fetch` downloads remote information and immediately merges it into the current branch.

B. `git pull` is essentially fetch followed by integration into the current branch.

C. `git push` updates your working tree from the remote repository.

D. Committing automatically sends the commit to the remote.

## Question 2: Remote-tracking branches and upstreams

A branch displays:

```text
* feature-parser 91ab2cd [origin/feature-parser: ahead 2, behind 1] Add parser
```

### (a) [1 mark]

What does `origin/feature-parser` represent?

### (b) [1 mark]

What does “ahead 2, behind 1” mean?

### (c) [1 mark]

Why can this status be stale?

## Question 3: Pull requests and conflicts

### (a) [1 mark]

Is a pull request a Git commit object stored in the repository? What is it instead?

### (b) [1.5 marks]

A push is rejected because a teammate pushed commits first. Describe a safe next step sequence.

### (c) [1.5 marks]

During conflict resolution, what should happen after manually editing the conflict markers out of a file?

## Question 4: Merge, rebase, and shared history

You are on `feature` and want to incorporate the latest `main` commits without adding a merge commit.

### (a) [1 mark]

Which command applies the latest `main` history beneath your feature commits?

### (b) [1.5 marks]

Explain one important difference between merging and rebasing.

### (c) [1 mark]

Why is rebasing pushed commits risky?

## Question 5: Finding and undoing regressions

### (a) [1 mark]

When is `git bisect` more useful than `git blame`?

### (b) [1.5 marks]

A bad commit has already been pushed to a shared `main` branch. Should you normally use `git reset` or `git revert`? Explain.

### (c) [1 mark]

What does `git bisect reset` do after the first bad commit is identified?
