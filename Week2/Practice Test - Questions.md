# Week 2 Practice Test: Shell Expansion, Scripting & Text Tools

Attempt the questions before opening the separate solutions file. Predict the shell's arguments and exit statuses before checking your answers.

## Question 1: Quoting and expansion

Let:

```bash
label="Week 2 Materials"
printf 'arg=<%s>\n' $label
printf 'arg=<%s>\n' "$label"
```

### (a) [1 mark]

How many output lines does each `printf` produce, and why?

### (b) [1 mark]

What does `echo '$label'` print, and why?

### (c) [1 mark]

Name one reason to quote a variable expansion by default.

## Question 2: Variables and exit status

### (a) [1 mark]

What is the difference between `COURSE=FIT2109` and `export COURSE=FIT2109` when a child process is started?

### (b) [1 mark]

What does `DEBUG=1 ./script.sh` do to the variable's lifetime?

### (c) [1.5 marks]

Why must `$?` be read immediately after the command whose status you care about?

## Question 3: Globs, regular expressions, and grep

### (a) [1 mark]

Explain the difference between `*.log` in `ls *.log` and `.*\.log` in `grep -E '.*\.log' names.txt`.

### (b) [1 mark]

What do these commands report differently?

```bash
grep -c 'ERROR' server.log
grep -q 'ERROR' server.log
```

### (c) [1.5 marks]

Write a command that recursively searches `src/` for `TODO` or `FIXME`, including line numbers and filenames.

## Question 4: Text processing and find

### (a) [1 mark]

Why is `sort -n` needed when sorting the values `10`, `2`, `1`, and `20` numerically?

### (b) [1 mark]

Why is `sort names.txt | uniq -c` safer than `uniq -c names.txt` for counting repeated names?

### (c) [1.5 marks]

Write a command that finds Python files under the current directory and searches them for `TODO` or `FIXME` with line numbers.

## Question 5: Safe shell scripting

A script contains:

```bash
cd "$backup_dir"
rm -rf *
```

### (a) [1 mark]

What is the danger if `cd` fails?

### (b) [1.5 marks]

Rewrite the two-line sequence so deletion does not continue after a failed directory change.

### (c) [1.5 marks]

Why is `cmd1 && cmd2 || cmd3` not always a safe replacement for an `if/else` statement?
