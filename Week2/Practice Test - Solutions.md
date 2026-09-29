# Week 2 Practice Test: Solutions and Learning Notes

## Question 1: Quoting and expansion

### (a)

The unquoted expansion produces three lines: `Week`, `2`, and `Materials`. Word splitting breaks the value at spaces before `printf` receives its arguments. The quoted expansion produces one line containing `Week 2 Materials` because double quotes preserve it as one argument.

### (b)

It prints `$label` literally. Single quotes prevent variable, command, arithmetic, and glob expansion.

### (c)

Quoting preserves spaces, tabs, and newlines in a value as one argument and prevents accidental glob expansion. This avoids commands receiving the wrong number of arguments or wrong filenames.

**Remember:** double quotes allow variable and command substitution while protecting word boundaries; single quotes make the text literal.

## Question 2: Variables and exit status

### (a)

`COURSE=FIT2109` creates a shell-local variable. A child process does not inherit it unless it is exported. `export COURSE=FIT2109` makes it part of the environment copied to child processes.

### (b)

It sets `DEBUG` only for the environment of that one command. The parent shell does not retain the variable after `./script.sh` exits.

### (c)

`$?` stores the exit status of the most recently completed command. Any later command, including `echo`, replaces it, so read it immediately:

```bash
some_command
status=$?
```

Zero conventionally means success; non-zero means failure.

## Question 3: Globs, regular expressions, and grep

### (a)

`*.log` is a shell glob expanded before `ls` runs, matching filenames ending in `.log`. `.*\.log` is a regular expression interpreted by `grep`; it matches text consisting of any characters followed by a literal `.log`.

### (b)

`grep -c` prints the number of matching lines. `grep -q` prints nothing and communicates whether a match exists through its exit status: zero for a match and one for no match.

### (c)

```bash
grep -rEn 'TODO|FIXME' src/
```

`-r` searches recursively, `-E` enables extended regular expressions, and `-n` includes line numbers. Quoting the pattern prevents the shell from interpreting it.

## Question 4: Text processing and find

### (a)

Plain `sort` compares text lexicographically, so `10` can appear before `2`. `sort -n` interprets the values numerically and orders them as `1`, `2`, `10`, `20`.

### (b)

`uniq` only detects adjacent duplicates. Sorting first puts equal names together, allowing `uniq -c` to count all occurrences rather than only consecutive repeats.

### (c)

```bash
find . -type f -name '*.py' -exec grep -nE 'TODO|FIXME' {} +
```

`find` selects files by filesystem properties; `grep` searches their contents. Quoting `*.py` prevents the shell from expanding the pattern before `find` receives it.

## Question 5: Safe shell scripting

### (a)

If `cd` fails, the shell remains in its previous directory. `rm -rf *` would then delete entries from that current directory, which may be unrelated to the intended backup directory.

### (b)

```bash
cd "$backup_dir" || exit 1
rm -rf -- ./*
```

The guard stops the script if the directory change fails. The explicit `./*` limits the wildcard to entries in the current directory, and `--` prevents a filename beginning with `-` from being treated as an option.

### (c)

In `cmd1 && cmd2 || cmd3`, `cmd3` runs if either `cmd1` fails or `cmd2` fails. In a real `if/else`, the else branch represents failure of the condition, not failure of an arbitrary command run inside the then branch. Use `if` when the branches have independent success conditions.

**Remember:** shell control flow depends on exit status, so guard dangerous operations and make failure paths explicit.
