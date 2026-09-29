# Week 7 Practice Test: Command-Line Environment

Attempt the questions before opening the separate solutions file. For command questions, explain which shell or process receives the effect.

## Question 1: Bash startup files

Which statement is TRUE?

Select one:

A. Every Bash shell reads `~/.bashrc`, including non-interactive scripts.

B. An interactive non-login shell normally reads `~/.bashrc`, while a non-interactive script uses `$BASH_ENV` when configured.

C. SSH login shells read only `~/.bashrc` and never read `/etc/profile`.

D. An alias placed in `~/.profile` is guaranteed to exist in every terminal opened later.

## Question 2: PATH and command lookup

A user creates an executable script at `~/bin/hello-fit2109` and runs:

```bash
export PATH="$HOME/bin:$PATH"
hello-fit2109
```

### (a) [1 mark]

Why does `hello-fit2109` work without typing its full path?

### (b) [1 mark]

What is the difference between running `hello-fit2109` and running `./hello-fit2109`?

### (c) [1 mark]

Name one command that can show how a command name resolves, and state what it helps diagnose.

## Question 3: Aliases, functions, and persistence

A user writes:

```bash
alias mkcd='mkdir -p "$1" && cd "$1"'
```

They want to run `mkcd reports/2026`.

### (a) [1 mark]

Why is an alias the wrong tool for this job?

### (b) [1.5 marks]

Write a suitable shell function that creates the directory and changes into it only if creation succeeds.

### (c) [1 mark]

Where could the function be placed so it is available in future interactive Bash shells?

## Question 4: Job control and persistent sessions

A long command is running in the foreground:

```bash
python3 analysis.py
```

### (a) [1.5 marks]

Describe the effect of pressing `Ctrl-Z`, then running `bg %1`, and later `fg %1`.

### (b) [1.5 marks]

Explain why this job-control sequence is not equivalent to running the command inside tmux when working over an unreliable SSH connection.

### (c) [1 mark]

Give commands to create a named tmux session called `analysis`, detach from it, and reattach later.

## Question 5: Diagnosing a shell environment

A script works when launched manually but fails from another terminal with `command not found` for a personal utility. The user says, “It is definitely installed.”

### (a) [1.5 marks]

Give two plausible environment causes based on Week 7.

### (b) [1.5 marks]

Give two commands or checks that would help distinguish those causes.

### (c) [1 mark]

Explain the difference between exporting a variable in the current shell and putting the export in a startup file.
