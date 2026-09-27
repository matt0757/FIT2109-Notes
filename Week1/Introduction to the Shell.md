# Week 1: Introduction to the Shell

## 1.1 History & Context

### Why "Terminal"?
- 1960s-70s: computers were huge/shared, accessed via **teletypes** (electric typewriters over cables), placed at the far ("terminal") end of the wire — hence the name.
- Teletypes → glass TTYs (screens replaced paper) → today's "terminal" is software emulating that old hardware (a **terminal emulator**).
- Linux kernel still uses the TTY abstraction (`/dev/tty`) even with no physical teletype.

### Unix & small tools philosophy
- 1969: Ken Thompson & Dennis Ritchie (Bell Labs) created **Unix** — many small programs, each doing one thing well, connected together (not one giant program).
- This is why piping `grep | sort | wc` works — 50-year-old design idea, still the shell's foundation.

### GNU, Linux, and the "GNU/Linux" name
- 1983: Richard Stallman starts **GNU** project — builds most CLI tools (`ls`, `grep`, `gcc`, etc.), free Unix-like OS.
- 1991: Linus Torvalds (Finnish student) writes the **Linux kernel**.
- GNU tools + Linux kernel = complete OS → "GNU/Linux." Opening a terminal on Ubuntu/Debian = using both.

### Bash's origin
- Original Unix shell = **Bourne shell (`sh`)**, Stephen Bourne, 1979 — defined pipelines, redirection, variables, scripting syntax.
- 1989: Brian Fox writes **Bash** (Bourne Again SHell) as a free GNU replacement — adds history, tab completion, job control.
- Became default on most Linux distros and macOS (until 2019, when macOS switched default to zsh).
- Other shells exist (zsh, fish, dash) — share core ideas. FIT2109 uses Bash specifically when exact shell matters.

## 1.2 What Is a Shell?
- Shell = a **program** that runs on Linux, acting as the command interface — reads what you type, interprets it, asks the OS to execute it.
- Analogy: kernel = engine (manages hardware/processes/files); shell = cockpit (programmable interface to operate the engine).
- Shells are swappable (bash/zsh/fish) — like swapping a steering wheel. You can also use Linux without a shell (via GUI), but shell = essential for automation/precision.

## 1.3 The Terminal Prompt
- Typical prompt: `student@fit2109:~/week1$ _`
  - Parts: **username** @ **hostname** : **working directory** **prompt char** (cursor)
- Prompt character: `$` for regular user, `#` for root/admin. You never type the `$` — it's just a marker that the shell is ready.

## 1.4 Essential Keys and Shortcuts
| Key | Does | Why it matters |
|---|---|---|
| ↑ / ↓ | scroll through command history | never retype a long command |
| Tab | auto-complete names | faster, catches typos |
| Ctrl+C | interrupt/kill running command | escape a stuck process |
| Ctrl+D | end of input / exit shell | signals "no more input" |
| Ctrl+L | clear the screen | same as `clear` |
| Ctrl+A | cursor to start of line | quick edits |
| Ctrl+E | cursor to end of line | quick edits |
| q | quit a pager (`man`, `less`) | escape a scrolling help page |

## 1.5 The Filesystem: One Tree from `/`
- Linux = single hierarchical tree, starting at root `/`. Everything (home folders, system programs, USB drives) is a branch beneath it.
- Different from Windows (separate trees per drive, `C:\`, `D:\`) — Linux **mounts** a USB drive as a folder inside the existing tree (e.g. `/media/usb`).

### Key directories
- `/home/` — user home directories (your files)
- `/usr/bin/` — most installed programs (`ls`, `grep`, etc.)
- `/etc/` — system-wide config files
- `/tmp/` — temporary files, cleared on reboot

## 1.6 Paths: How to Point at a File
- **Absolute path**: starts from `/`, full route — e.g. `/home/student/week1/notes.txt`. Same meaning anywhere (like a full postal address).
- **Relative path**: starts from current location — e.g. `week1/notes.txt` or `../data`. Meaning depends on where you are ("turn left at the corner").

### Special path shortcuts
- `~` = home directory (e.g. `/home/student`). `cd ~` always goes home.
- `.` = current directory. `./script.sh` = "run script.sh from here."
- `..` = parent directory. `cd ..` goes up one level.

## 1.7 Working Directory and File Manipulation
- **Working directory** = where the shell currently is. Any file command uses it as the starting point unless you give an absolute path.

### Core commands
```bash
pwd                 # print working directory
ls -la              # list all files with details

mkdir week1         # make a directory
touch notes.txt     # create empty file

cp notes.txt backup.txt     # copy
mv notes.txt summary.txt    # rename/move
rm backup.txt               # delete (NO undo!)
```
- **Caution**: `rm` deletes immediately — no Trash, no undo. Always double-check the path.

### Glob patterns (wildcards)
| Pattern | Matches | Example |
|---|---|---|
| `*` | any string (0+ chars) | `*.txt` — all .txt files |
| `?` | exactly one character | `log?.txt` — log1.txt, log2.txt |
| `[abc]` | one char from the set | `report[12].txt` — report1.txt or report2.txt |

```bash
ls *.txt
cp *.log backup/
rm *.tmp    # careful — no undo
```
- **Important**: the SHELL expands globs, not the command — `ls *.txt` becomes `ls notes.txt other.txt ...` before `ls` even runs. `ls` never sees the `*` itself.

### Reading `ls -la` output
```
drwxr-xr-x  3 student staff  96 May 20 10:15 week1
-rw-r--r--  1 student staff 142 May 20 10:12 notes.txt
```
- Columns: type+permissions, link count, owner, group, size, date, name.
- First character: `d` = directory, `-` = file, `l` = symbolic link.

## 1.8 Permissions
- Every file/dir has permissions for 3 user classes: **owner**, **group**, **others** — each with 3 bits: **read (r)**, **write (w)**, **execute (x)**.
- `r=4, w=2, x=1, -=0`. Octal digit = sum of bits (e.g. `rwx`=7, `r-x`=5).
- Common combos: `755` = `rwxr-xr-x` (owner full, others read+exec), `644` = `rw-r--r--`, `600` = `rw-------` (owner only).

### `chmod` — two styles
```bash
# Octal (numeric): set all at once
chmod 755 script.sh    # rwxr-xr-x
chmod 600 secret.txt   # rw-------

# Symbolic: change specific bits
chmod u+x script.sh    # add execute for owner
chmod o-rw secret.txt  # remove read+write from others
```

### Example
```bash
touch foo.txt
ls -l foo.txt          # -rw-r--r--
chmod 600 foo.txt
ls -l foo.txt          # -rw-------
chmod u+x foo.txt
ls -l foo.txt          # -rwx------
```

## 1.9 Scripts and the Execute Bit
- A shell script = plain text file of shell commands. To run it, need TWO things:
  1. **Shebang** on line 1: `#!/bin/bash` (tells OS which interpreter to use).
  2. **Execute bit** set: `chmod u+x script.sh`.

```bash
echo '#!/bin/bash' > hello.sh
echo 'echo "Hello FIT2109!"' >> hello.sh
chmod u+x hello.sh
./hello.sh              # Hello FIT2109!
```
- Note `./hello.sh` — the `./` is required, or the shell won't find it (explained via PATH next).

## 1.10 $PATH and How the Shell Finds Programs
- Typing `ls` → shell searches directories listed in `$PATH` (colon-separated) to find the program.
```bash
echo "$PATH"      # see your PATH
which ls          # find where ls lives
type cd           # cd is a shell builtin, not in PATH
```
- **Why `cd` isn't in `which`**: `cd` is a **shell builtin** — part of the shell itself, not a separate program. It must be a builtin because it changes the shell's OWN current directory; an external program runs in its own process and exits without being able to affect the shell's directory.

## 1.11 Pipes and Redirection
- Every program has 3 standard streams: **stdin** (input), **stdout** (normal output), **stderr** (error output).
- Default: stdin = keyboard, stdout/stderr = terminal. Redirection/pipes change where these go.

### Redirection (to/from files)
```bash
echo "first line" > notes.txt     # write (overwrite)
echo "second line" >> notes.txt   # append
sort < notes.txt                  # read from file via stdin
```
- `>` overwrites, `>>` appends.

### Pipelines (chain commands)
```bash
grep "ERROR" server.log | sort | uniq -c | sort -nr
```
- Each command's stdout feeds the next command's stdin — no intermediate files needed. Data flows left to right, like an assembly line.

### Example
```bash
echo -e "banana\napple\ncherry\napple" > fruit.txt
cat fruit.txt
sort fruit.txt | uniq -c
#    2 apple
#    1 banana
#    1 cherry
```
- Redirection = routes streams to/from files. Pipes = chain commands together.