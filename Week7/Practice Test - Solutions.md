# Week 7 Practice Test: Solutions and Learning Notes

## Question 1: Bash startup files

**Answer: B.**

Interactive non-login shells normally read `~/.bashrc`. Non-interactive scripts use the file named by `$BASH_ENV` when that variable is set. Interactive login shells read `/etc/profile` and then the first available login startup file such as `~/.bash_profile` or `~/.profile`.

A login file often sources `~/.bashrc` so interactive customisation is shared, but that is a configuration choice rather than an automatic rule.

**Common trap:** assuming “opened a terminal” and “logged in through SSH” are the same shell-startup path.

**Remember:** first classify the shell as login/non-login and interactive/non-interactive, then identify its startup files.

## Question 2: PATH and command lookup

### (a)

Adding `$HOME/bin` to the front of `PATH` tells Bash to search that directory when a command is entered without a slash. Because the script is executable and the directory is in `PATH`, Bash can resolve `hello-fit2109` to that file.

### (b)

`hello-fit2109` asks Bash to search the command-resolution order: functions, builtins, and then `PATH` directories. `./hello-fit2109` contains a slash, so Bash skips `PATH` and runs the exact file in the current directory.

### (c)

`type -a hello-fit2109` shows aliases, functions, builtins, and executable matches. `command -v hello-fit2109` gives a concise resolution. `echo "$PATH"` shows the directories being searched. These checks help diagnose different PATH values, shadowed commands, or missing executable permissions.

## Question 3: Aliases, functions, and persistence

### (a)

Aliases perform simple textual substitutions and do not properly accept positional arguments such as `$1`. A command needing arguments and conditional logic should be a shell function.

### (b)

```bash
mkcd() {
  mkdir -p "$1" && cd "$1"
}
```

`&&` means `cd` runs only if `mkdir -p` succeeds. Quoting `"$1"` preserves directory names containing spaces.

### (c)

Place the function in `~/.bashrc` for future interactive non-login Bash shells. If login shells are used, ensure the login startup file sources `~/.bashrc`, or place appropriate login setup in `~/.bash_profile` or `~/.profile`.

**Common trap:** putting an interactive alias or function only in a file that the current shell type does not read.

**Remember:** aliases are shortcuts; functions are small programs.

## Question 4: Job control and persistent sessions

### (a)

`Ctrl-Z` suspends the foreground job and returns control to Bash. `bg %1` resumes job 1 in the background. `fg %1` brings that job back to the foreground so its input and terminal interaction return to the shell's foreground job.

### (b)

Job control belongs to the current shell session. If the SSH connection or terminal closes, the shell and its jobs may receive a hangup or otherwise stop being usable. tmux keeps a server-side session alive after detaching; the user can reconnect later and reattach to the same panes and processes.

### (c)

```bash
tmux new -s analysis
# detach from inside tmux with Ctrl-b d
tmux attach -t analysis
```

`tmux ls` can list existing sessions. A convenient repeatable command is `tmux new -A -s analysis`, which attaches if the session exists or creates it otherwise.

## Question 5: Diagnosing a shell environment

### (a)

Plausible causes include:

- The utility's directory is not in `PATH` in the failing shell.
- The export was placed in `~/.profile`, but the terminal is a non-login shell that reads `~/.bashrc` instead.
- The script is non-interactive and does not receive the user's interactive startup customisation.
- The shell has a different environment, interpreter, or command resolution than the manual terminal.

### (b)

Useful checks include:

```bash
echo "$PATH"
command -v hello-fit2109
type -a hello-fit2109
echo "$0"
shopt login_shell
echo "$-"
```

The first three inspect PATH and command lookup. The last three help identify how the shell was invoked and whether it is interactive or a login shell.

### (c)

`export PATH="$HOME/bin:$PATH"` changes the environment of the current shell and child processes started from it. Putting that command in a startup file makes the change happen automatically for future shells that read that file; it does not retroactively change already-running shells.

**Remember:** “installed” is not the same as “resolvable by this shell.” Always inspect the environment that actually runs the command.
