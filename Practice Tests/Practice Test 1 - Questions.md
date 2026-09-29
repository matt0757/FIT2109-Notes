# FIT2109 Practice Test 1: Questions

This practice test follows the style of the SEB mock test and is based on Weeks 1–8.

## Question 1: Shell pipelines and quoting

You have a file named `deploy log.txt` and want to print only the lines containing `ERROR`, followed by the number of matching lines. Which command does this correctly?

Select one:

A. `grep "ERROR" deploy log.txt | wc -l`

B. `grep "ERROR" "deploy log.txt" | wc -l`

C. `grep "ERROR" "deploy log.txt" > wc -l`

D. `wc -l | grep "ERROR" "deploy log.txt"`

## Question 2: Editors and tooling

Which statement is FALSE?

Select one:

A. In Vim, `i` enters Insert mode and `Esc` returns to Normal mode.

B. A VS Code buffer can contain unsaved changes that are not yet present in the file on disk.

C. Formatting and linting are identical because both tools only change the layout of source code.

D. VS Code Search looks across a project, while Find searches within the current file.

## Question 3: Reproducible environments

Consider this Dockerfile:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN python -m pip install -r requirements.txt
COPY . .
CMD ["python", "main.py"]
```

Which statement is FALSE?

Select one:

A. `requirements.txt` must be available in the Docker build context for the first `COPY` to work.

B. The `RUN` instruction installs dependencies while the image is being built.

C. Editing `main.py` on the host automatically changes an existing image, even if the image is not rebuilt.

D. `CMD` supplies the default command for a container created from the image.

## Question 4: Git history and repository state

A teammate runs:

```text
$ git log --oneline --graph --decorate --all
*   7ac91de (HEAD -> main) Merge branch 'feature-cli'
|\
| * 3b82f10 (feature-cli) Add CLI help
* | 5d4a2cc Fix README example
|/
* 18e7b44 Add project skeleton
* 02fd991 (tag: v1.0) Initial commit
```

### (a) [1 mark]

At which commit did the two lines of work diverge? Provide the commit hash only.

### (b) [2 marks]

Explain what `HEAD -> main` and `tag: v1.0` identify in this graph.

### (c) [1 mark]

Is this a fast-forward merge or a true merge? Briefly explain.

The teammate then runs `git status`:

```text
On branch main
Changes to be committed:
  modified:   app.py

Changes not staged for commit:
  modified:   app.py
  modified:   README.md

Untracked files:
  notes.md
```

### (d) [1.5 marks]

Why can `app.py` appear in both the staged and unstaged sections?

## Question 5: Persistent command-line sessions

A data-processing command takes 40 minutes on a remote server. You are connected through SSH and the connection may drop. A colleague runs:

```bash
tmux new -s analysis
python process.py > process.log 2>&1
```

They then detach from tmux and close the SSH connection.

### (a) [2 marks]

Explain what tmux provides and what happens to `process.py` after the SSH connection closes.

### (b) [1.5 marks]

Why would adding `&` to the Python command alone not provide the same protection?

### (c) [1.5 marks]

Contrast `tmux` with shell job control commands such as `Ctrl-Z`, `bg`, and `fg`.

## Question 6: Systematic debugging

A Python program fails with this traceback:

```text
Traceback (most recent call last):
  File "report.py", line 24, in <module>
    total = average(values)
  File "report.py", line 9, in average
    return sum(values) / len(values)
ZeroDivisionError: division by zero
```

### (a) [1.5 marks]

Using the traceback-reading approach, identify the exception and the first line of your code you would investigate.

### (b) [2 marks]

Use the RIP model to explain why this bug is only visible for some inputs.

### (c) [1.5 marks]

Describe a disciplined next step before changing the code. Your answer should mention reproduction, inspecting the actual state, and changing one thing at a time.

### (d) [1 mark]

Name one tool or technique from Weeks 3 or 8 that could help inspect this failure, and explain what evidence it would provide.
