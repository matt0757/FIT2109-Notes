# Week 2: Shell Expansion, Scripting & Text Tools (Simplified)

## 2.1 How the Shell Evaluates a Command
- What you type is NOT what runs directly — Bash transforms it through 7 ordered expansion stages.

| # | Expansion | What happens | Example |
|---|---|---|---|
| 1 | Tilde expansion | `~` → `$HOME`; `~user` → that user's home | `cd ~/projects` → `cd /home/student/projects` |
| 2 | Parameter/variable expansion | `$name`/`${name}` replaced by value | `echo $HOME` |
| 3 | Command substitution | `$(cmd)` replaced by cmd's output | `echo "Today is $(date +%A)"` |
| 4 | Arithmetic expansion | `$((expr))` evaluated as integer math | `echo $((2+3))` → 5 |
| 5 | Word splitting | unquoted expansion result split on spaces/tabs/newlines ($IFS) | `dir="my files"; ls $dir` → 2 args |
| 6 | Filename (glob) expansion | unquoted glob replaced with matching filenames | `ls *.txt` |
| 7 | Quote removal | quotes stripped before command runs — program never sees them | `ls "my file.txt"` → receives `my file.txt` |

- **Key pitfall**: word splitting (5) happens AFTER variable expansion (2) — an unquoted variable with spaces splits into multiple args. `ls $dir` vs `ls "$dir"` can differ completely.

### Seeing word splitting
```bash
label="Week 2 Materials"
printf 'argument: <%s>\n' $label     # 3 lines: <Week> <2> <Materials>
printf 'argument: <%s>\n' "$label"   # 1 line: <Week 2 Materials>
```
- Rule: unquoted expansion CAN be split at separators; double quotes preserve it as ONE argument.

### Arithmetic expansion
```bash
echo $((2 + 3))         # 5
echo $((10 / 3))        # 3 (integer division, rounds toward zero)
count=4; echo "Next: $((count + 1))"   # Next: 5
errors=0; errors=$((errors + 1))       # increment pattern
```

### Command substitution
```bash
echo "Today is $(date +%A)"
files=$(ls)
echo $files      # unquoted: newlines collapse to spaces
echo "$files"    # quoted: newlines preserved
```
- Note: most common real-world Bash bugs are quoting, word splitting, path management, command options, return values, and error handling (per a 2022 study of open-source Bash scripts).

## 2.2 Quoting Rules
- Three quoting modes:

| Mode | Syntax | What happens | When to use |
|---|---|---|---|
| No quotes | `$name`, `*.txt` | vars & globs expand, word-split on whitespace | rarely — only when you WANT splitting/globbing |
| Double quotes | `"$name"` | vars & command subs expand, spaces preserved, globs do NOT expand | almost always — safe default |
| Single quotes | `'literal'` | everything literal, no expansion at all | truly literal strings (regex, JSON, `$` signs) |

### Comparison
```bash
file="Week 2 Materials/errors log.txt"
ls $file          # word-splits into 4 broken args — fails
ls "$file"        # one argument, works correctly
echo '$file'      # literally prints: $file (no expansion)
echo "*.txt"      # glob does NOT expand inside double quotes
echo *.txt        # glob DOES expand (unquoted)
```

- **Default rule**: always quote variables — `cp "$source" "$dest"`, `grep "$pattern" "$logfile"`.
- Deliberately unquoted cases: glob expansion (`ls *.log`), or splitting a list (`for f in $list` — fragile, prefer separate arguments).
- Mixing styles: `echo "$HOME"/notes.txt` and `echo "It's a shell"` (literal `'` inside double quotes) both work fine.
- **Common trap**: inside `[ ]` test conditions, an unquoted empty variable breaks syntax:
```bash
if [ "$name" == "Alice" ]; then ...   # safe
if [ $name == "Alice" ]; then ...     # breaks if $name is empty
```

## 2.3 Variables and the Environment
- Shell variable = only visible in current shell. `export` = also copied into child processes.
- Distinction is determined by `export`, NOT by uppercase/lowercase naming (that's just convention).

| Pattern | Current shell | Child processes | Lifetime |
|---|---|---|---|
| `LOCAL_COURSE=FIT2109` | usable | do NOT receive it | until unset/shell exits |
| `export SHARED_COURSE=FIT2109` | usable | receive a copy | until unset/shell exits |
| `TEMP_COURSE=FIT2109 env` | not kept | only that one command | for that command only |

- Why child starts empty: shell copies its exported-variable environment into the child; shell-local vars are never copied.

### Useful patterns
| Pattern | Meaning | Example |
|---|---|---|
| `name=value` | assign shell variable (no spaces around `=`) | `dir=/tmp/work` |
| `export name` | mark for export | `export PATH` |
| `export name=value` | assign + export in one step | `export EDITOR=vim` |
| `NAME=value cmd` | set var for ONE command only | `DEBUG=1 ./script.sh` |
| `env` | list all env vars | `env \| grep PATH` |
| `${var:-default}` | use default if unset/empty | `name=${1:-world}` |

- **No spaces around `=`**: `course = FIT2109` is a syntax error — shell tries to RUN a command called `course`.

### Command substitution recap
```bash
today=$(date +%F)
echo "Backup name: logs-$today.tar"
echo "This folder has $(ls | wc -l) entries"
```
- Quote it: `"$(command)"` — output word-splits just like an unquoted variable.

### Common variables
| Variable | Kind | Holds |
|---|---|---|
| `$HOME` | env var | home directory path |
| `$PATH` | env var | colon-separated command search dirs |
| `$USER` | env var | username |
| `$PWD` | env var (shell-maintained) | current working directory |
| `$SHELL` | env var | login shell path |
| `$?` | special param | exit status of last command (not exported) |

## 2.4 Exit Status and Command Chaining
- Every command returns an **exit status** on exit: `0` = success, non-zero (1–255) = failure (different tools use different codes).
- `$?` holds the last command's exit status — read it IMMEDIATELY (it's overwritten by every subsequent command, even `echo`).

### Conventional exit codes
| Code | Meaning | Example cause |
|---|---|---|
| 0 | success | normal completion |
| 1 | general error | `mkdir existing_dir` |
| 126 | found but not executable | missing execute bit |
| 127 | command not found | typo/not in PATH |
| 128+N | terminated by signal N | 130 = Ctrl+C (signal 2) |

### Chaining operators
| Operator | Meaning | Use |
|---|---|---|
| `cmd1 && cmd2` | run cmd2 only if cmd1 succeeded | chain dependent steps |
| `cmd1 \|\| cmd2` | run cmd2 only if cmd1 failed | fallback / bail out |
| `cmd1 ; cmd2` | always run cmd2 | independent sequence |

### Most important pattern: bail out on failure
```bash
cd "$target_dir" || exit 1
```
- **Silent cd failure trap**: if `cd "$backup_dir"` fails silently (no `|| exit 1` guard), a following `rm -rf *` deletes the CURRENT directory's contents instead of the backup's. Always guard `cd` this way.
- `grep -q "ERROR" app.log && echo "Errors found"` — quiet exit-status-only check.

### Important caveat
- `cmd1 && cmd2 || cmd3` is NOT a safe if/else — if `cmd2` fails, `cmd3` ALSO runs (because `||` sees the combined status). Use a real `if` statement (section 2.9) for reliable branching.

## 2.5 Shell Globs vs Regular Expressions
- Look similar, but are completely different languages interpreted by different software at different times.

| | Shell glob | Regular expression |
|---|---|---|
| Interpreted by | the shell, before command runs | the tool (grep/find/sed), after shell passes args |
| Matches | filenames in the filesystem | text content / strings |
| Context | `ls *.log` | `grep 'pattern' file` |

### Glob syntax
| Glob | Matches |
|---|---|
| `*` | any string (not `/`) |
| `?` | exactly one character |
| `[abc]` | one char from the set |
| `[0-9]` | one digit |
| `**` | any path incl. subdirs (needs `globstar` in Bash) |

### Regex syntax (basics)
| Regex | Matches |
|---|---|
| `.` | any single character |
| `.*` | any string (zero+ of `.`) |
| `[0-9]+` | one+ digits (needs `-E`) |
| `^ERROR` | line starting with ERROR |
| `\.log$` | line ending in `.log` |
| `WARN(ING)?` | WARN or WARNING (needs `-E`) |

- **Common mistake**: glob `*.log` means "any filename ending .log"; regex `*` means "zero+ of preceding element" — `*.log` as regex is often a syntax error (nothing precedes the `*`). Regex for "any chars then .log" = `.*\.log`.
- Always quote regex patterns: `grep -E 'pattern'`, not `grep -E pattern` (shell would try to glob-expand it).

## 2.6 Searching Text with `grep`
- `grep` searches for lines matching a pattern and prints them.

### Essential flags
| Flag | Meaning | Use |
|---|---|---|
| `-E` | extended regex (`+`, `?`, `\|`, `()`) | almost always use this |
| `-i` | case-insensitive | varying-case logs |
| `-n` | line numbers | jump to right line |
| `-o` | print only matched text | extract values |
| `-q` | quiet, exit status only | scripts |
| `-r` | recursive directory search | find across files |
| `-v` | invert match | filter out noise |
| `-c` | count of matches | how many errors |
| `-h` | suppress filename prefix | multi-file pipelines |

### Common patterns
```bash
grep '^ERROR' server.log            # anchors: start of line
grep 'failed$' app.log              # end of line
grep -E 'WARN(ING)?' server.log     # optional group
grep -Eo 'ERROR [0-9]+' server.log  # extract matched text
grep -rEn 'TODO|FIXME' src/         # recursive + line numbers
grep -q 'ERROR' app.log && echo "errors found"   # quiet check

grep -E 'colou?r' docs.txt          # ? = optional char
grep -rl 'TODO' src/                 # -l = filenames only
grep -A 2 'ERROR' app.log            # 2 lines After
grep -B 1 'ERROR' app.log            # 1 line Before
grep -C 2 'ERROR' app.log            # 2 lines Context (both)
grep -e 'ERROR' -e 'WARN' app.log   # multiple patterns (OR)
```

### Exit status
- `0` = at least one match found. `1` = no match. `2` = error (e.g. file not found).
- Always quote the pattern: `grep -E 'WARN(ING)?'` — unquoted, the shell tries to glob-expand `(ING)?`.

## 2.7 Text Processing: `sort`, `uniq`, `cut`
- `grep` selects lines; `cut`/`sort`/`uniq` shape them: cut = columns, sort = order, uniq = collapse/count duplicates.

| Tool | Does | Key flags |
|---|---|---|
| `sort` | sorts lines (alphabetical default) | `-n` numeric, `-r` reverse, `-u` dedupe, `-t` delimiter, `-k` key field |
| `uniq` | removes/counts ADJACENT duplicates (pipe `sort` first!) | `-c` count, `-d` only dupes, `-i` ignore case |
| `cut` | extracts columns | `-d` delimiter, `-f` field(s), `-c` char position |

### cut examples
```bash
cut -d',' -f2 enrolments.csv        # single field
cut -d',' -f1,3 enrolments.csv      # multiple fields
cut -d',' -f2- enrolments.csv       # field 2 to end
cut -f2 data.tsv                    # tab-separated (default delimiter)
cut -d':' -f1 /etc/passwd           # extract usernames
cut -c1-19 app.log                  # by character position
```

### sort examples
```bash
printf '10\n2\n1\n20\n' | sort       # WRONG for numbers: 1,10,2,20
printf '10\n2\n1\n20\n' | sort -n    # CORRECT: 1,2,10,20
sort -nr numbers.txt                # largest first
sort -t',' -k3,3 -f enrolments.csv  # sort CSV by 3rd field, case-insensitive
sort -u names.txt                   # dedupe in one step
```
- **Common mistake**: bare `sort` on numbers sorts digit-by-digit (lexicographic) — always use `-n` for numeric data.

### uniq examples
```bash
sort names.txt | uniq        # remove adjacent dupes (must sort first!)
sort names.txt | uniq -c     # count occurrences
sort names.txt | uniq -d     # only lines appearing more than once
```
- `uniq` only removes ADJACENT duplicates — sort first or scattered dupes survive.

### Full pipeline examples
```bash
cut -d',' -f2 enrolments.csv | sort | uniq -c | sort -nr      # count units, most enrolled first
grep -Eo 'ERROR [0-9]+' server.log | sort | uniq -c | sort -nr # rank error codes by frequency
cut -c1-16 app.log | sort | uniq -c | sort -nr | head -5       # busiest minutes
```

## 2.8 Finding Files with `find`
- `find` locates files by filesystem properties (name/type/permissions/size/mtime) — NOT by content (that's `grep`'s job).
```
find [starting-directory] [expression]
```

### Essential predicates
| Predicate | Meaning |
|---|---|
| `-name '*.log'` | filename matches glob (ALWAYS quote it) |
| `-type f` / `-type d` / `-type l` | file / directory / symlink |
| `! -perm -u=x` | NOT executable by owner (`!` negates) |
| `-size +1M` | larger than 1MB (`k`/`M`/`G` units) |
| `-newer ref.txt` | modified more recently than ref.txt |
| `-maxdepth 2` | search depth limit |
| `-regex 'pattern'` | full PATH matches regex (not just filename) |
| `-exec cmd {} +` | run cmd on results, `{}` = filename, `+` batches |

### Examples
```bash
find . -type f -name '*.py'                              # all python files
find . -type f -name '*.sh' ! -perm -u=x                 # non-executable shell scripts
find . -maxdepth 1 -type f -name '*.log'                 # one level deep
find . -type f -name '*.py' -exec grep -nE 'TODO|FIXME' {} +
find . -type f -mtime -7                                  # modified in last 7 days
find . -type f -mtime +30                                 # NOT modified in 30+ days
find . -type f -size +1M
find . -type f -empty                                      # zero-size files
find . -type f \( -name '*.py' -o -name '*.sh' \)          # OR condition
find . -type f -name '*.c' ! -path '*/build/*'             # exclude a directory
find . -type f -name '*.py' | wc -l                        # count matches
```
- `-mtime N`: `-` prefix = more recent than N days ago; `+` prefix = older than N days ago. Use `-mmin` for minutes.
- **`-regex` matches the FULL path**, not just filename — `find . -regex 'error.log'` won't match `./logs/error.log`; need `'.*/error\.log'`.
- Always quote the `-name` glob: `find . -name '*.py'`, not `find . -name *.py` (shell would expand it first).

### -exec vs xargs
```bash
find . -name '*.log' -exec wc -l {} +     # batches args, usually preferred
find . -name '*.log' -exec echo "Processing: {}" \;   # runs once per file
find . -name '*.log' -print0 | xargs -0 wc -l          # equivalent via xargs
```

### Combining find + grep + sort
```bash
find . -type f -name '*.log' -exec grep -h 'ERROR' {} + | sort | uniq -c | sort -nr
```
- `-h` suppresses grep's filename prefix so only matching lines flow into sort/uniq.

## 2.9 Shell Scripting Basics
- A script = plain text file of shell commands, run with the same rules as typing at the prompt.

### Making a script runnable
1. **Shebang** on line 1: `#!/usr/bin/env bash` (portable — finds bash via PATH, works on macOS + Linux).
2. **Execute bit** set: `chmod u+x script.sh`.
```bash
#!/usr/bin/env bash
echo "Hello from a script"
```
```bash
chmod u+x greet.sh
./greet.sh
```
- Shebang only means something at the top of a SCRIPT FILE — typing it at an interactive prompt does nothing useful (ignored as a comment in Bash, errors in Zsh).

### Positional arguments
| Variable | Value |
|---|---|
| `$1, $2, …` | positional arguments |
| `$#` | number of arguments |
| `$@` | all arguments as separate words (quote as `"$@"`) |
| `$0` | script's own name |
| `${1:-default}` | use `$1` if set, else default |

```bash
#!/usr/bin/env bash
name=${1:-world}
echo "Hello, $name"
```

### Conditionals
```bash
#!/usr/bin/env bash
file="$1"
if [ -f "$file" ]; then
    echo "File exists: $file"
    wc -l < "$file"
else
    echo "File not found: $file" >&2
    exit 1
fi
```

| Test | True when |
|---|---|
| `-f "$path"` | regular file exists |
| `-d "$path"` | directory exists |
| `-e "$path"` | exists (any type) |
| `-z "$str"` | string empty |
| `-n "$str"` | string non-empty |
| `"$a" = "$b"` | strings equal |
| `"$a" != "$b"` | strings not equal |
| `"$a" -eq "$b"` | numbers equal (also `-ne`, `-lt`, `-gt`) |

### Loops
```bash
for file in "$@"; do
    if [ -f "$file" ]; then
        echo "$file: $(wc -l < "$file") lines"
    else
        echo "$file: not found" >&2
    fi
done

for log in logs/*.log; do
    echo "Processing $log"
    grep -c 'ERROR' "$log" || echo "  no errors"
done
```

### Functions
```bash
warn() {
    echo "warning: $1" >&2
}
warn "missing input"
```
- Inside a function, `$1` refers to the function's own first argument, not the script's.
- Write errors to stderr (`>&2`) so they don't contaminate stdout output in a pipeline.

## 2.10 Writing Reliable Scripts
- A script that runs without crashing ≠ a script that behaves correctly. Three key habits.

### Safe script header
```bash
#!/usr/bin/env bash
set -u
set -o pipefail
```

### Habit 1: `set -u` — fail on unset variables
- Without it, a typo'd variable name silently expands to empty (e.g. `rm -rf "$trget_dir/"` → `rm -rf /` if the real var is empty).
- With `set -u`: Bash errors immediately on the typo.

### Habit 2: `set -o pipefail` — catch pipeline failures
- By default, a pipeline's exit status = only the LAST command's status — a failing earlier stage is silently ignored.
```bash
grep 'ERROR' missing.log | wc -l; echo $?    # 0 — misleading!
set -o pipefail
grep 'ERROR' missing.log | wc -l; echo $?    # 2 — grep's real error code
```

### Habit 3: quote every variable
- Any variable that might contain spaces/newlines/glob chars must be quoted everywhere.

### Common script problems to spot
1. `$1` unquoted on assignment (safe there, but inconsistent style).
2. `[ -f $file ]` unquoted — breaks on empty/spaced values.
3. `grep ERROR $file` unquoted — word-splits filenames with spaces.
4. No `set -u` — silent wrong behavior on missing arguments.
5. No `exit 1` in the else branch — caller using `&&` thinks it succeeded.
6. Error message to stdout instead of stderr — should be `echo "..." >&2`.
- Use **shellcheck.net** to catch remaining issues.

## 2.11 SSH and Remote Access
- SSH encrypts a connection to run commands on a remote machine. Key-based auth replaces passwords.

### Key pair
| | Private key | Public key |
|---|---|---|
| Location | `~/.ssh/id_ed25519` (never share) | `~/.ssh/id_ed25519.pub` (copy to servers) |
| Role | signs a challenge to prove identity | server verifies signature; stored in `~/.ssh/authorized_keys` |

- Handshake: server sends challenge → client signs with private key → server verifies with public key → no password sent.

### Generating and copying a key
```bash
ssh-keygen -t ed25519 -C "your@email.com"   # modern, recommended over RSA
chmod 600 ~/.ssh/id_ed25519                  # SSH refuses overly-permissive private keys

ssh-copy-id -i ~/.ssh/id_ed25519.pub user@hostname
# manual alternative:
cat ~/.ssh/id_ed25519.pub | ssh user@hostname 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

### Connecting and running commands
```bash
ssh user@hostname                              # interactive shell
ssh user@hostname 'pwd && uname -a'            # single remote command
result=$(ssh user@hostname 'ls /var/log/*.log | wc -l')
```
- Single-quote the remote command to keep it literal locally — the REMOTE shell expands it.

### Transferring files
```bash
scp deploy.sh user@hostname:/tmp/              # local → remote
scp user@hostname:/var/log/app.log .           # remote → local
scp -r src/ user@hostname:~/project/           # recursive directory copy
```

### Interactive transfer with sftp
```bash
sftp user@hostname
sftp> ls          # remote list
sftp> put file    # upload
sftp> get file    # download
sftp> quit
```

- Why key-based auth matters for scripts: passwords can't be typed by automation — key auth works silently (no interaction needed), which is why CI/CD, deploy scripts, and cron jobs use keys.

## 2.12 Comparing Files with `diff`
- `diff` compares two text files line by line, reports what changes would make them identical. Powers `git diff` and automated tests.

### Basic (default) output
```bash
diff file1.txt file2.txt
# 2c2
# < banana
# ---
# > blueberry
```
- `2c2` = line 2 changed. `<` = left file, `>` = right file.

### Exit status (key for scripting)
- `0` = identical, `1` = differ, `2` = error.

### Unified format (`-u`) — same as `git diff`
```bash
diff -u file1.txt file2.txt
# --- file1.txt
# +++ file2.txt
# @@ -1,3 +1,3 @@
#  apple
# -banana
# +blueberry
#  cherry
```
- `-`/`+` lines = removed/added; unprefixed lines = unchanged context.

### diff as a test tool
```bash
./my_script.sh input.txt > actual.txt
diff expected.txt actual.txt              # no output = match

diff -q expected.txt actual.txt && echo "PASS" || echo "FAIL"
diff -u expected.txt actual.txt || echo "--- output did not match ---"
```

### Process substitution `<(cmd)` — Bash only, no temp file needed
```bash
diff expected.txt <(./my_script.sh input.txt)
diff <(./old_version.sh data.txt) <(./new_version.sh data.txt)
```
- Runs `cmd` and exposes its output as a file-like path (named pipe) — avoids writing a temp file.

### Other useful diff options
| Option | Does |
|---|---|
| `-u` | unified format (default choice) |
| `-q` | quiet — only says whether they differ |
| `-i` | ignore case |
| `-w` | ignore all whitespace differences |
| `-b` | ignore leading/trailing whitespace changes |
| `-y` | side-by-side display |
| `-r` | recursively compare directories |

- Watch for trailing newline mismatches causing unexpected diffs — use `-b` to debug.

## 2.13 Week 2 Cheatsheet: Tools at a Glance

| Situation | Reach for | Quick start |
|---|---|---|
| Quoting/logic bug in script | shellcheck | `shellcheck script.sh` or shellcheck.net |
| Understand an unfamiliar command | explainshell.com | paste the command |
| Build/test a regex interactively | regex101.com | select flavour, paste pattern |
| Faster example-first docs | tldr | `tldr grep`, `tldr find`, `tldr ssh` |
| Authoritative reference | man | `man grep` → `q` to quit |
| Script does the wrong thing silently | `bash -x` | `bash -x script.sh` (prints each command) |
| Check loaded SSH keys | `ssh-add` | `ssh-add -l` |
| Public key accepted but login fails | check permissions | `chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys` |
| Check script output vs expected | `diff` | `diff expected.txt <(./script.sh input)` |
| See exactly what changed | `diff -u` | `diff -u old.txt new.txt` |