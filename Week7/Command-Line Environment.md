# Week 7: Command-Line Environment

## 7.1 Shell Initialisation and Startup Files
- Bash doesn't start the same way every time — 3 shell types, each reads different startup files.

| Shell type | Reads |
|---|---|
| Interactive **login** shell | `/etc/profile`, then first of `~/.bash_profile`, `~/.bash_login`, `~/.profile` |
| Interactive **non-login** shell | `~/.bashrc` |
| Non-interactive (scripts) | `$BASH_ENV` variable |

- Example: opening a terminal emulator on Ubuntu = usually non-login shell → reads `~/.bashrc`. SSH login = login shell → reads `/etc/profile` then `~/.bash_profile`/`~/.profile`.
- Common pattern: `~/.bash_profile` sources `~/.bashrc`, so login shells still get interactive settings:
```bash
# ~/.bash_profile
if [ -f ~/.bashrc ]; then
  . ~/.bashrc
fi
```

### Checks to explore this
```bash
echo "$0"          # how the shell was invoked
shopt login_shell   # is this a login shell?
echo "$-"           # includes "i" if interactive
```

### Safer mental model
- `~/.bashrc` → interactive shell customisation (aliases, prompt).
- `~/.bash_profile` / `~/.profile` → login-time setup.
- `BASH_ENV` → non-interactive (script) initialisation, only when needed.
- Common mistake: putting an alias only in `~/.profile` — won't show up in a plain non-login terminal that only reads `~/.bashrc`.

## 7.2 PATH Resolution, Environment Variables & Command Lookup
- Typing a command name (no slash) → Bash checks in order: **shell function** → **builtin** → directories in `$PATH`.
- Command with a **slash** (e.g. `./myscript.sh`) → Bash skips PATH, runs that exact file.
- `cd` is a builtin (handled directly by Bash, not searched in PATH) — that's why it behaves differently.
- Bash caches found executables in a hash table (doesn't re-search PATH every time).

### Diagnostic commands
```bash
echo "$PATH"        # list of directories Bash searches
type -a ls           # shows if name = builtin/function/alias/executable
type -a cd
command -v python3   # find where a command resolves to
command -v git
```
- Useful for "works on one machine but not another" problems.

### Environment variables
- Env = name-value pairs inherited by child processes (e.g. `PATH`, `HOME`, `SHELL`, `EDITOR`).
- A shell variable is local to that shell **unless exported** — exporting makes it visible to child processes.

### Example: add personal scripts dir to PATH
```bash
mkdir -p "$HOME/bin"
echo 'echo "Hello from custom command"' > "$HOME/bin/hello-fit2109"
chmod u+x "$HOME/bin/hello-fit2109"
export PATH="$HOME/bin:$PATH"
hello-fit2109
```
- This only lasts for the current shell. To persist: put the `export PATH=...` line in a startup file (`~/.bashrc` or `~/.profile`).

## 7.3 Dotfiles and Command-Line Configuration
- **Dotfile** = config file starting with `.` — hidden unless you use `ls -a`.
- Examples: `~/.bashrc`, `~/.bash_profile`, `~/.profile` (shell startup); `~/.inputrc` (Readline/line-editing config, used when `INPUTRC` unset); `~/.screenrc` (GNU Screen); `~/.tmux.conf` (tmux, uses its own syntax, not shell script).

### Minimal `~/.bashrc` example
```bash
# ~/.bashrc
export PATH="$HOME/bin:$PATH"
alias ll='ls -lah'
PS1='\u@\h:\w\$ '
```
- `PS1` = "Prompt String 1" — controls what you see before typing a command.

| Symbol | Meaning |
|---|---|
| `\u` | username |
| `@` | literal @ |
| `\h` | hostname |
| `:` | literal colon |
| `\w` | current working directory |
| `\$` | `$` for normal user, `#` for root |

- Example result: `alice@ubuntu:~/projects$ `

### Minimal `~/.inputrc` example
```
set editing-mode vi
set show-all-if-ambiguous on
```
- Lets you set editing style (vi vs default/Emacs) separately from shell startup.

### Why dotfiles matter
- Enable **repeatable personal environments** — keep them in a small repo, reuse across machines (reproducibility theme).
- Good practice: keep dotfiles readable, commented, minimal — don't pile on random internet snippets you don't understand.

## 7.4 Customising the Shell
- Two main customisation tools: **aliases** (simple substitutions) and **functions** (support arguments/logic).

### Aliases
```bash
alias ll='ls -lah'
alias gs='git status'
```
- Created with `alias`, removed with `unalias`.
- **Aliases can't take arguments** — for anything needing a parameter, use a function instead.

### Functions
```bash
mkcd() {
  mkdir -p "$1" && cd "$1"
}
```
- Functions are real shell code — can use arguments, conditionals, multiple commands. Sit between aliases and full scripts.

### Prompt customisation (PS1)
```bash
PS1='\u@\h:\w\$ '
# richer:
PS1='[\A] \u@\h:\w jobs=\j\$ '
```
- `\A` = 24-hour time (HH:MM), `\j` = number of active jobs.
- Prompt = functional info display (where you are, who you are, background jobs), not just decoration.

### Tooling integration
- Line-editing behavior configurable via `~/.inputrc` (Readline).
- tmux's prompt can use vi-style keys if `VISUAL`/`EDITOR` is set to a vi-like editor.
- Good customisation = a few aliases + a couple functions + a readable prompt + consistent editing mode. Keep it useful, not noisy.

## 7.5 Job Control & Managing Long-Running Processes
- **Job control** = ability to stop, resume, and move processes between foreground/background.
- Bash tracks a job per pipeline; `jobs` shows the table of currently running jobs.

### Start a command in the background
```bash
sleep 300 &
jobs
```
- Bash prints a job number + process ID when started with `&`.

### Suspend and resume workflow
```bash
python3 long_task.py
# press Ctrl-Z         → suspends foreground process, returns control to Bash
jobs
bg %1                  # resume job 1 in the background
jobs
fg %1                  # bring job 1 back to the foreground
```
- `%1` = jobspec referring to job number 1 (`%1` alone acts like `fg %1`).

### Killing a job
```bash
kill %1
```
- `kill` can target a process ID or a jobspec. Without a signal specified, sends `SIGTERM`.

- Summary: job control = first-level process management **within one shell session**, before reaching for tmux/screen.

## 7.6 Persistent Terminal Sessions: tmux & screen
- Job control doesn't survive a dropped SSH connection or closed terminal — **terminal multiplexers** solve this.
- **tmux**: runs multiple terminal programs inside one terminal; sessions can be **detached** and **reattached** later (same or different terminal).
- **GNU Screen**: similar — full-screen window manager multiplexing a terminal between processes; programs keep running even when detached.

### tmux model
- Structure: **panes** (where programs run) → belong to **windows** → grouped into **sessions**.
- A session can be attached to a client or detached (running in background).

### Minimal tmux workflow
```bash
tmux new -s work        # create new session named "work"
# inside tmux, prefix = Ctrl-b
# Ctrl-b d              → detach (session keeps running)
tmux ls                 # list sessions
tmux attach -t work     # reattach
```
- Handy form: `tmux new -A -s work` — attaches if session exists, else creates it (good for repeatable remote workflows).

### Common tmux key bindings (prefix = Ctrl-b)
| Keys | Action |
|---|---|
| `Ctrl-b c` | new window |
| `Ctrl-b %` | split pane left/right |
| `Ctrl-b "` | split pane top/bottom |
| `Ctrl-b` + arrow keys | move between panes |
| `Ctrl-b n` / `Ctrl-b p` | next/previous window |
| `Ctrl-b d` | detach |

### Minimal Screen workflow
```bash
screen -S work           # start named session
# inside screen: Ctrl-a d  → detach
screen -ls                # list sessions
screen -r work             # reattach
```

### Summary
- **Job control** = manages processes inside the current shell.
- **tmux/screen** = manage persistent sessions that survive beyond the current terminal/connection.
- Together: practical toolkit for long builds, remote log monitoring, and multi-pane work.