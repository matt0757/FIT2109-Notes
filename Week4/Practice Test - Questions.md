# Week 4 Practice Test: Git Foundations

Attempt the questions before opening the separate solutions file. Draw the repository state when a question describes multiple areas or commits.

## Question 1: Git areas and staging

A repository contains a committed `app.py`. You edit it, run `git add app.py`, and edit it again before committing.

### (a) [1.5 marks]

Which versions are represented in the working tree, staging area, and last commit?

### (b) [1 mark]

Why can `git status` list `app.py` as both staged and modified but unstaged?

## Question 2: Commits and branches

Which statement is TRUE?

Select one:

A. A branch is a full independent copy of the repository history.

B. A commit is only a diff and has no parent or metadata.

C. A branch is a movable label pointing to a commit, while a commit records a snapshot and metadata.

D. Every merge creates a commit with exactly one parent.

## Question 3: Reading history

Given:

```text
*   a8f2c11 (HEAD -> main) Merge feature-search
|\
| * 6bc11d0 (feature-search) Add search command
* | 31a92ef Update documentation
|/
* 0ef44a1 Add project skeleton
* 7d1c008 (tag: v1.0) Initial commit
```

### (a) [1 mark]

At which commit did the branches diverge?

### (b) [1 mark]

Is the merge fast-forward or a true merge? Explain.

### (c) [1 mark]

What does the tag `v1.0` identify?

## Question 4: Ignoring and recovering files

### (a) [1 mark]

A secret file is already tracked, and you add `secret.env` to `.gitignore`. Will Git stop tracking the existing file automatically?

### (b) [1 mark]

What command removes the file from tracking while leaving the local file in place?

### (c) [1.5 marks]

When is `git stash` useful, and what does it not do?

## Question 5: Safe commit history

### (a) [1 mark]

What is the purpose of `git diff --staged` before a commit?

### (b) [1 mark]

When is `git commit --amend` appropriate, and what is its main collaboration risk?

### (c) [1.5 marks]

Why should a developer avoid committing secrets even if they delete the secret in a later commit?
