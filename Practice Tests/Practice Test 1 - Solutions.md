# FIT2109 Practice Test 1: Solutions

## Question 1: Shell pipelines and quoting

**Answer: B.**

`"deploy log.txt"` keeps the filename as one argument because the space is inside double quotes. `grep` writes matching lines to standard output, and the pipe sends them to `wc -l`, which counts those lines.

- A is split into two filenames: `deploy` and `log.txt`.
- C redirects output to a file or command name rather than piping it to `wc`.
- D puts the commands in the wrong order and does not provide the filename to `grep` correctly.

This tests Week 1 pipes and Week 2 quoting and word splitting.

## Question 2: Editors and tooling

**Answer: C.**

Formatting concerns layout such as spacing and indentation. Linting checks for suspicious or problematic code such as unused imports, unreachable code, or risky patterns. They are complementary tools, not identical tools.

- In Vim, `i` enters Insert mode and `Esc` returns to Normal mode.
- A VS Code buffer may differ from the saved file until it is saved.
- Find is file-local, while Search can inspect the whole project.

This tests Week 3.

## Question 3: Reproducible environments

**Answer: C.**

An image contains the files captured during its build. Editing `main.py` on the host does not modify an existing image. The image must be rebuilt, or the file must be supplied through a runtime mount.

- `COPY` reads from the build context, so `requirements.txt` must be inside that context.
- `RUN` is a build-time instruction.
- `CMD` is the default run-time command for a container.

This tests Week 6 distinctions between a Dockerfile, image, and container.

## Question 4: Git history and repository state

### (a)

`18e7b44`

The two branches both point back to `18e7b44` before taking different paths.

### (b)

- `HEAD -> main` means `HEAD` is currently attached to the `main` branch, and `main` points to commit `7ac91de`.
- `tag: v1.0` means the fixed tag `v1.0` points to commit `02fd991`. A tag is a name for a specific commit and does not move like a branch normally does.

### (c)

This is a **true merge**. The merge commit `7ac91de` has two parents, one from `main` and one from `feature-cli`, so the histories had diverged.

### (d)

`app.py` was staged at one version and then edited again. The staged section describes the snapshot currently in the index for the next commit; the unstaged section describes newer working-tree changes to the same file. `git add app.py` would stage the newer changes as well.

This tests Week 4 Git objects, branches, tags, staging, and merge history.

## Question 5: Persistent command-line sessions

### (a)

`tmux new -s analysis` creates a named tmux session. The Python process runs inside that session, and the session can be detached while its processes continue running. When the SSH connection closes, the tmux session and `process.py` remain on the server. You can later reconnect and run `tmux attach -t analysis` to view the process and its output.

The redirection sends normal output and error output to `process.log`, which also makes later inspection easier.

### (b)

`&` only puts the command in the background within the current shell session. It does not create a persistent terminal session or automatically protect the process from the terminal and SSH connection closing. Depending on how the process handles the connection's hangup signal, it may terminate when SSH disconnects.

### (c)

`Ctrl-Z` suspends a foreground job. `bg` resumes that job in the background, and `fg` brings it back to the foreground. These are shell job-control operations tied to the current shell session. `tmux` provides a persistent session that can be detached and reattached after the terminal or SSH client disappears.

This tests Week 7 job control and tmux.

## Question 6: Systematic debugging

### (a)

The exception is `ZeroDivisionError: division by zero`. The first line of this code to investigate is line 9 in `average`, where `sum(values)` is divided by `len(values)`. The traceback's bottom line gives the error type and message, and the nearest frame in the program's own code shows where it was raised.

### (b)

The faulty operation is only visible when all three RIP conditions hold:

- **Reachability:** the call to `average(values)` must execute.
- **Infection:** `values` must be empty, making `len(values)` equal to zero.
- **Propagation:** the invalid division must reach the program's visible failure rather than being handled or replaced later.

For a non-empty list, the line is reachable but the denominator is not zero, so the infection condition does not hold and no exception appears. For an empty list, all three conditions hold.

### (c)

First reproduce the failure with the exact input, working directory, interpreter/package versions, and command that triggered it. Then inspect the actual value of `values` and its length at the failure point, using a breakpoint, debugger, or focused logging. Form a hypothesis about why an empty list reached `average`, change one thing only, and rerun the same reproduction steps plus related tests.

### (d)

A VS Code breakpoint could pause before the division and show `values` and `len(values)` in the Variables or Debug Console panels. Alternatively, `logging` at the `DEBUG` level could record the input size without permanently relying on ad-hoc `print` statements. A Python linter or type checker could also flag related suspicious code, although it may not detect this input-dependent runtime failure directly.

This tests Week 8 traceback reading, RIP, reproduction, evidence-based debugging, logging, and debugger tooling, with the Week 3 editor/debugger concepts.
