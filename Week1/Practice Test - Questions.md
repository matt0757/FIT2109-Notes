# Week 1 Practice Test: Introduction to the Shell

Attempt the questions before opening the separate solutions file. For command questions, explain what the shell and the operating system each do.

## Question 1: Paths and working directories

You are currently in `/home/student/projects/fit2109`.

### (a) [1 mark]

Which type of path is `/home/student/notes/week1.md`? Which type is `../notes/week1.md`?

### (b) [1 mark]

What path does `~/week1/notes.md` refer to if the user's home directory is `/home/student`?

### (c) [1 mark]

Why can `notes.md` refer to different files after changing directories?

## Question 2: Files, directories, and safe operations

A directory contains `report.txt`, `.draft`, `data.csv`, and a subdirectory named `backup`.

### (a) [1 mark]

Which command lists all entries with detailed information, including `.draft`?

### (b) [1 mark]

Write a command that copies `report.txt` into `backup/` without deleting the original.

### (c) [1 mark]

What is the main danger of `rm` compared with deleting a file through a desktop Trash folder?

## Question 3: Permissions and scripts

A script has this permission string:

```text
-rw-r-----
```

### (a) [1 mark]

State the permissions of the owner, group, and others.

### (b) [1 mark]

What does `chmod 750 deploy.sh` allow the owner, group, and others to do?

### (c) [1.5 marks]

A script begins with `#!/usr/bin/env bash` but running `./deploy.sh` produces “Permission denied”. Give the command that fixes the likely permission problem and explain why the shebang alone was insufficient.

## Question 4: PATH and builtins

### (a) [1.5 marks]

Explain why `cd` must be a shell builtin rather than an ordinary external program.

### (b) [1 mark]

What does `./hello.sh` communicate to Bash that `hello.sh` does not?

## Question 5: Streams, redirection, and pipes

### (a) [1 mark]

Explain the difference between `>` and `>>`.

### (b) [1.5 marks]

What does this command do?

```bash
grep "ERROR" server.log | sort | uniq -c | sort -nr
```

### (c) [1 mark]

Name the three standard streams and give their usual defaults in a terminal.
