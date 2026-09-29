# Week 1 Practice Test: Solutions and Learning Notes

## Question 1: Paths and working directories

### (a)

`/home/student/notes/week1.md` is an **absolute path** because it starts at the filesystem root `/`. `../notes/week1.md` is a **relative path** because it is interpreted from the current working directory and moves to its parent first.

### (b)

It refers to `/home/student/week1/notes.md`. `~` expands to the current user's home directory.

### (c)

A relative path starts from the shell's current working directory. After `cd`, the same relative name can point to a different location. `pwd` shows the current working directory.

**Remember:** absolute paths are location-independent; relative paths depend on where the shell currently is.

## Question 2: Files, directories, and safe operations

### (a)

```bash
ls -la
```

`-l` requests the long listing and `-a` includes hidden entries such as `.draft`.

### (b)

```bash
cp report.txt backup/
```

`cp` creates a copy while leaving the source file in place.

### (c)

`rm` deletes immediately and normally provides no desktop Trash or undo step. Check the path carefully before using it, especially with wildcards or recursive options.

## Question 3: Permissions and scripts

### (a)

For `-rw-r-----`:

- Owner: read and write.
- Group: read only.
- Others: no permissions.

The leading `-` means it is a regular file.

### (b)

`750` means:

- Owner: `rwx` (read, write, execute).
- Group: `r-x` (read and execute).
- Others: `---` (no access).

The digits use `r=4`, `w=2`, and `x=1`.

### (c)

```bash
chmod u+x deploy.sh
```

The shebang identifies the interpreter to use once the file is executed; it does not set the filesystem execute bit. `chmod u+x` grants execute permission to the owner.

**Common trap:** confusing “contains executable instructions” with “is marked executable by the filesystem.”

## Question 4: PATH and builtins

### (a)

`cd` must change the current directory of the existing shell. An external program runs in a child process; when it exits, the parent shell's directory would be unchanged. A builtin can modify the shell's own state directly.

### (b)

`./hello.sh` contains a slash, so Bash runs that exact file in the current directory and does not search `PATH`. `hello.sh` asks Bash to find a command with that name through its command lookup rules, and the current directory is not normally in `PATH`.

## Question 5: Streams, redirection, and pipes

### (a)

`>` writes output to a file and overwrites existing contents. `>>` appends output to the end of the file.

### (b)

The pipeline:

1. selects lines containing `ERROR`;
2. sorts them;
3. counts adjacent identical lines with `uniq -c`;
4. sorts the counts numerically in descending order with `sort -nr`.

Each command's standard output becomes the next command's standard input.

### (c)

- **stdin:** input; normally the keyboard.
- **stdout:** normal output; normally the terminal.
- **stderr:** error output; normally the terminal.

Redirection and pipes change where these streams go.

**Remember:** redirection connects a command to files; a pipe connects one command's output to another command's input.
