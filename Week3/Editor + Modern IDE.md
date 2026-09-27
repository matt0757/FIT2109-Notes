# Week 3: Editors — VS Code, Vim & Terminal Editors (Simplified)

## 3.1 What Is an Editor, Really?
- An editor isn't just "where you type code" — it's where you read unfamiliar files, compare code, search definitions, make precise changes, run formatting, inspect diagnostics, and interact with terminals/VCS/remote systems.
- Good editor habits = faster AND more accurate work.

### Two families of editor
- **Terminal editors**: run inside a shell session. Fast start, work well over SSH, keyboard-centred. Good for remote/minimal systems.
- **GUI editors**: richer project views, panes, mouse support, extensions, language tooling. Better for larger multi-file workflows.
- Neither is universally "right" — different trade-offs for different situations.

## 3.2 Terminal Editors: nano, Vim, and Emacs

### nano — small and friendly
- Beginner-friendly: undo/redo, search-and-replace, auto-indent, line numbers, on-screen shortcut hints.
- Preinstalled on many Unix systems (incl. macOS). No modal interface to learn first.
- `nano shell.sh` → start editing almost immediately.

### Vim — modal editing
- **Modal**: text entry and commands are SEPARATE modes.
  - **Normal mode**: keys = commands.
  - **Insert mode**: keys = text.
- Efficient editing = navigating/selecting/repeating/transforming with intent, not just typing fast.
- This unit only expects: open a file, switch modes, move around, make small edits, save, quit (see 3.6/3.7).

### Emacs — an extensible environment
- Described as advanced, extensible, customizable, self-documenting.
- Can control subprocesses, auto-indent programs, show multiple files at once, edit remote files as if local.
- Illustrates: some editors are extensible WORKING ENVIRONMENTS, not just file editors.

- **Note**: strong editor opinions ("correct" editor) = professional identity, not proof one tool is objectively best.

## 3.3 GUI Editors and VS Code
- GUI editors add: windows, menus, tabs, file explorers, split panes, mouse support (in addition to shortcuts).
- Main advantage: visibility & integration — manage multiple files, see project structure, access diagnostics/refactoring, switch between editing/searching/running with less friction.

### VS Code layout
- Project-oriented UI: **Explorer** (left, project files), **Editor area** (main, open files), sidebars, panel (output/integrated terminal), multiple editor groups side by side.
- Most programming work is multi-file — VS Code is built for comparing files, searching a folder, tracing definitions, inspecting warnings.

## 3.4 Buffers vs Files: What "Editing" Really Means
- A file exists on disk. An editor session holds an in-memory editing state = a **buffer**, which may differ from what's saved.
- The file and buffer are DIFFERENT until you save (`:w` in Vim, Ctrl+S in VS Code).
- This distinction explains: unsaved-changes dots, autosave, open editors persisting across sessions, diff views.

### How VS Code makes this visible
- **Explorer**: project's folders/files on disk.
- **Editor area**: currently open files.
- **"Open Editors" region**: what's loaded into the editing session.
- **Editor groups**: arrange open items side by side.
- VS Code remembers workspace state (open files, layout) between launches.
- **Concept check answer**: closing without saving loses the BUFFER, not the file on disk. VS Code's "hot exit" is a deliberate exception — it preserves unsaved buffers across close/reopen.

## 3.5 Modes, Selections, and Workspace Layout

### Mode: what a keypress means depends on context
- Vim = clearest example (Insert vs Normal mode).
- VS Code is less visibly modal, but meaning of a keypress still depends on WHERE focus is (editor, terminal, Command Palette, search field, Explorer). "Where am I in the interface" is still part of editing fluency.

### Selection
- More than mouse-dragging — represents a phrase to replace, a block to reindent, several repeated occurrences to edit at once, or a region for a refactor. VS Code emphasizes multiple selections (see 3.12).

### Terms to keep distinct
- **File** = persistent object on disk.
- **Tab** = visible UI representation of an open editor item.
- **Pane / editor group** = a region that can hold one or more open items.
- **Workspace / opened folder** = the broader project context.
- Side-by-side editing = two PANES (editor groups), not two copies of one file.

### Language mode
- VS Code assigns a **language mode** per file (often inferred from extension) — enables symbol navigation, completion, formatting, refactoring. A file is "text interpreted through language-specific tooling."

## 3.6 Vim Basics: Modes and Essential Commands
```bash
vim file.txt    # opens in Normal mode — keypresses are commands, not text
```

| Key | Effect |
|---|---|
| `i` | Insert text before cursor |
| `a` | Append text after cursor |
| `o` | Open new line below, start typing |
| `Esc` | Leave Insert mode → Normal mode (always safe, doesn't undo anything) |

### Saving and quitting (command-line mode, starts with `:`)
| Command | Effect |
|---|---|
| `:w` | Save |
| `:q` | Quit (refuses if unsaved changes) |
| `:wq` | Save and quit |
| `:q!` | Quit WITHOUT saving |

- Goal for this unit: functional enough to read, move, insert, save, exit — not full mastery.
- To learn properly: run `vimtutor` in terminal (~30 min, ships with Vim).

## 3.7 Vim Motions, Operators & Survival Cheat Sheet

### Basic motions
| Key | Moves |
|---|---|
| `h` `j` `k` `l` | left / down / up / right |
| `w` | start of next word |
| `b` | start of current/previous word |
| `e` | end of current/next word |
| `0` | beginning of line |
| `^` | first non-blank character |
| `$` | end of line |
| `gg` | top of file |
| `G` | end of file |

### Editing commands
| Key | Effect |
|---|---|
| `x` | delete char under cursor |
| `dd` | delete whole line |
| `yy` | yank (copy) whole line |
| `p` | paste after cursor |
| `u` | undo |
| `Ctrl+r` | redo |

### Operator + motion (core Vim idea)
- `dw` = delete from cursor to start of next word.
- `d$` = delete from cursor to end of line.
- Pattern: **what to do** (operator) + **how far** (motion) — composable, not memorized case by case.

### Searching
- `/pattern` + Enter = jump to next match. `n` = repeat forward, `N` = repeat backward. Search is itself a motion — composes with operators.

### Minimum survival set
```
Open a file:         vim file.txt
Insert text:         i / Append: a / New line: o
Return to Normal:    Esc
Move:                h j k l | By words: w b e | Line start/end: 0 $
Delete:              x  dd  dw
Copy/paste:          yy  p
Undo/Redo:           u  Ctrl+r
Search:              /pattern  n  N
Save/quit:           :w  :q  :wq  :q!
```
- To go further: `vimtutor`, openvim.com, VIM Adventures (game), `:help motion.txt`.
- Reasonable goal: not abandoning VS Code for Vim — just being unbothered on a server with no GUI editor. (VS Code even has a Vim extension.)

## 3.8 VS Code Basics: Opening, Editing, and Saving

### Opening a folder (not just a file)
- VS Code is project-oriented: `File > Open Folder…`, or `code .` from a terminal in that directory.
- macOS: may need "Shell Command: Install 'code' command in PATH" from Command Palette first.
- **Windows/WSL note**: run `code .` from your Ubuntu terminal (not PowerShell) so VS Code opens in WSL mode — check bottom-left corner for "WSL: Ubuntu". Opening from Windows side instead gives you the wrong terminal/interpreter (PowerShell, missing python3, etc.) with no obvious error.

### Creating/opening a file
- Explorer → New File icon, or right-click → New File.
- `Ctrl+N` (Cmd+N) for untitled editor, save later.
- To open existing file: click in Explorer, or Quick Open (`Ctrl+P`) to jump by name.

### Editing — no mode to switch
- Unlike Vim: click anywhere and type, no Normal/Insert distinction.

### Saving
- `Ctrl+S` (Cmd+S). Unsaved file = filled dot on tab (buffer≠file, from 3.4).
- `File > Auto Save` to skip manual saving.

### Command Palette — the escape hatch
- `Ctrl+Shift+P` (Cmd+Shift+P, or F1) — type what you want in plain words ("format", "toggle terminal"). Fuzzy-matches.
- Shows keyboard shortcut next to each matching command (if one exists) — doubles as shortcut discovery tool.

### Minimum survival set
| Action | Shortcut |
|---|---|
| Open a folder | File > Open Folder… (or `code .`) |
| New file | Ctrl+N |
| Save | Ctrl+S |
| Save As | Ctrl+Shift+S |
| Quick Open (jump to file) | Ctrl+P |
| Command Palette | Ctrl+Shift+P |
| Toggle integrated terminal | Ctrl+\` (same on macOS) |
| Close current editor | Ctrl+W |

## 3.9 Tooling: Completion, Diagnostics, Formatting vs Linting
- Language tooling (built-in + marketplace extensions) adds: snippets, completion, IntelliSense, linters, debuggers.
- Shortens the feedback cycle — catch small issues before they compound.

### Formatting vs linting — different jobs
| | Formatting | Linting |
|---|---|---|
| Answers | "How should this LOOK?" | "Does anything look SUSPICIOUS?" |
| Concerned with | layout, spacing, indentation, style consistency | unreachable code, unused imports, risky patterns |
| Won't catch | logical problems (can look clean but be broken) | doesn't reformat anything |

- Both are complementary, not competing.

### Where language support comes from
- Often from dedicated extensions, not built-in. Example — Python needs 4 separate pieces:
  - **Python** extension — base support, finds/runs interpreter.
  - **Pylance** — language service (completion, types, symbol navigation).
  - **Black Formatter** — the formatter.
  - **Pylint** — the linter.
- Consequence: same issue may be reported twice in different words (formatter vs linter vs language service each have their own opinion).
- If everything (completion/diagnostics/formatting) breaks at once → usually means the editor can't find a Python interpreter.

### Code Actions
- Quick Fixes and refactorings surfaced through Code Actions; some can run automatically on save (e.g. organize imports) — more reliable than manual multi-place edits.
- Team benefit: shared formatters/linters → cleaner diffs, more consistent practice.

## 3.10 Navigating a Project: Explorer, Quick Open, Search
- Three navigation tools answer three different questions:

| You know... | Use | Shortcut |
|---|---|---|
| the file's NAME | Quick Open | Ctrl+P / Cmd+P |
| some TEXT in it | Search | Ctrl+Shift+F / Cmd+Shift+F |
| neither, want project shape | Explorer | Ctrl+Shift+E / Cmd+Shift+E |

- If you're scrolling a folder tree for something whose name you already know → use Quick Open instead.

### Explorer
- Project-shaped view of filesystem: dirs, files, open editors, create/rename/move/delete. Pairs with editor groups (open one file, split, open a related file beside it).

### Quick Open
- Fuzzy-matches filenames (no need for whole name or consecutive letters). Think in terms of "likely target" ("the settings file," "the main entry point") instead of visual hunting.

### Find vs Search
- **Find** (`Ctrl+F`/`Cmd+F`) = within the current file.
- **Search** (`Ctrl+Shift+F`/`Cmd+Shift+F`) = across the whole project.
- Two different questions: "where in THIS file?" vs "where anywhere in the PROJECT?" Good first move when reading unfamiliar code (search for `main`, `TODO`, a config key, an error message string).

## 3.11 Structured Navigation and Keyboard Habits
- Beyond file/text search: **minimap** (high-level overview + jumps), **sticky scroll** (shows enclosing scope lines while scrolling), **breadcrumbs** (file path + symbol path — where you are structurally, not just which file).

### Semantic (language-aware) navigation
| Action | Shortcut | Answers |
|---|---|---|
| Go to Definition | F12 | "where is this defined?" |
| Find All References | Shift+Alt+F12 / Shift+Option+F12 | "where is this used?" |
| Go to Symbol in File | Ctrl+Shift+O / Cmd+Shift+O | "what's in this file, take me to one" |

- Cursor must be on the name first — all three act on what the cursor is inside.
- **Outline view** (side bar) = same info as Go to Symbol, shown as a permanent list.
- **Key distinction**: Find All References returns only genuine symbol USES (language-aware); a text search for the same string also matches comments/strings/longer names containing it. Same reasoning makes Rename Symbol (3.13) safer than replace-all.

### Keyboard-centred habits
- Reducing keyboard↔mouse switching (Card et al. 1980's "homing" cost) measurably improves efficiency — not about raw speed, but keeping attention on reasoning about code rather than operating the editor.

## 3.12 Multi-File Editing: Split Views and Multi-Cursor
- Real work is multi-file: source + test + config + build script. VS Code supports many editors open side by side (vertically/horizontally), editor groups holding stacks of items.
- **Compare before changing**: keep related files visible simultaneously (e.g. renaming a function, or aligning a test with implementation) — reduces "changed one place, forgot the related one."

### How to split the editor
| Method | How | Best for |
|---|---|---|
| Split Editor | Ctrl+\ (Cmd+\), or split icon in tab bar | splitting the file you're already viewing |
| Open to the Side | Ctrl+Enter (Cmd+Enter) after Quick Open selection, or Alt/Option+click in Explorer | opening a DIFFERENT file into a new group directly |
| Drag a tab | drag to an edge of the editor area | choosing exactly where the group lands (bottom = horizontal split) |

- `Ctrl+1/2/3` (Cmd+1/2/3) = jump focus to group 1/2/3. `Ctrl+W` (Cmd+W) = close active editor; closing last editor in a group removes the split.

### Multi-cursor and multi-selection
- Useful for repeated edits: renaming config lines, adding same prefix, adjusting similar calls. Value = not just speed, but visual control (see which places are affected before committing).

| Method | Shortcut | Best for |
|---|---|---|
| Alt+Click | Alt/Option+click | placing cursors at specific unrelated spots |
| Add Selection to Next Find Match | Ctrl+D / Cmd+D | select occurrences ONE AT A TIME (can skip unwanted ones) |
| Select All Occurrences | Ctrl+Shift+L / Cmd+Shift+L | change EVERY match at once (same blind-spot risk as search-and-replace) |
| Insert Cursor Above/Below | Ctrl+Alt+↑/↓ / Cmd+Option+↑/↓ | editing a vertical column of similar lines |
| Column (box) selection | Shift+Alt+drag / Shift+Option+drag | selecting a rectangular block across lines |

- `Esc` collapses back to a single cursor.
- Key distinction: `Ctrl+D` = incremental (skip unwanted matches); `Ctrl+Shift+L` = all at once (faster, no filtering — can't distinguish a variable from an identical word in a comment/string).

### Column selection
- Selects a RECTANGLE regardless of the text content — good for adding the same prefix/suffix to a block of otherwise-different lines. `Shift+Alt+drag` (Shift+Option+drag on macOS).

## 3.13 Refactoring: Search-and-Replace vs Language-Aware Tools
- Search-and-replace = quick for the same textual pattern across many files, BUT: text matches don't necessarily equal one programming concept (may hit comments, docs, fixtures, unrelated contexts). Always preview/verify before replace-all.

### Refactoring = editing program STRUCTURE, not text
- Provided by language services; same UI/commands across languages (exact refactorings depend on tooling).
- **Rename Symbol** (`F2`, all platforms): put cursor on the name, type new name — updates every GENUINE reference across the project, and ONLY those. A text replace-all would also hit comments, docstrings, and longer identifiers containing the string.
- **Code Actions on save** (e.g. auto-organize imports) — partial automation once trusted; improves team consistency.

### The decision ladder
| Situation | Tool |
|---|---|
| Tiny, local change | Edit directly |
| Several nearby places, same obvious change | Multi-cursor (3.12) |
| Many plain-text occurrences | Search-and-replace, with careful preview |
| Structural change language tooling understands | Refactoring command / code action (Rename Symbol = F2) |

## 3.14 Debugging in the Editor
- Editor + language tooling can show a program's LIVE STATE while running (not just static reading) — replaces most print-statement guesswork.

### Breakpoints
- A marked line where execution pauses if reached. Click in the gutter next to the line number, start under the debugger (Run and Debug view, not a normal run). Program freezes at that point — variables hold their momentary values.

### Reading a paused program
- **Variables pane**: names/values in current scope.
- **Call Stack pane**: chain of function calls that led here ("how did we get here").
- **Watch pane**: your own tracked expressions, updated as you step.

### Stepping controls
| Control | Shortcut | Effect |
|---|---|---|
| Continue | F5 | run to next breakpoint |
| Step Over | F10 | run current line, don't enter called functions |
| Step Into | F11 | enter the called function |
| Step Out | Shift+F11 | finish current function, return to caller |

- Choosing step-over vs step-into is itself a skill — stepping into everything gets overwhelming fast.

### Conditional breakpoints and logpoints
- **Conditional breakpoint**: pauses only when your expression is true (e.g. bug only on the 100th iteration).
- **Logpoint**: prints a message to console when hit, WITHOUT pausing — no print statement to add/remove from source.
- **Debug Console**: REPL scoped to the paused state — evaluate expressions live.
- Getting a debugger to start needs a launch config + often a language-specific debugger extension (same dependency pattern as 3.9's linting/formatting).
- This page = mechanics of the tool. Week 8 = the mindset of debugging (hypothesis-driven, MREs, systematic reasoning) built on top of these mechanics.

## 3.15 Remote Editing with VS Code
- A modern editor can develop on non-local systems (container, remote machine, WSL) as a full dev environment. Benefits: same OS as deployment, isolated dev env, consistent contributor environment, tools unavailable locally, access from multiple machines.

### The extension family
- **Remote - SSH**: opens a folder on a remote machine/VM running an SSH server.
- **Dev Containers**: opens a folder inside a container (no SSH server needed).
- **WSL extension**: same idea for Windows Subsystem for Linux.
- Editor stays local (familiar VS Code UI); files/tooling live/run remotely.

### Key gotcha: SSH server requirement
- Remote-SSH needs the TARGET machine already accepting SSH connections. Your own laptop is usually NOT an SSH target by default (macOS Remote Login off, Ubuntu/WSL don't install openssh-server by default, Windows' SSH server needs admin rights + is optional).

### Windows: two separate `ssh` programs
- Remote-SSH extension runs on the WINDOWS side, uses Windows' own `ssh`, reading `C:\Users\<Username>\.ssh\config`.
- The `ssh` used since Week 2 lives INSIDE Ubuntu/WSL, reads a separate `~/.ssh/config`.
- Hosts/keys configured in one are invisible to the other.

### Windows: WSL extension is essential, not optional
- Running `code .` from the Ubuntu terminal puts VS Code into WSL remote mode — window runs on Windows, but files/terminal/extensions run in Ubuntu.
- Bottom-left indicator reads "WSL: Ubuntu" when connected — check this before running anything; most confusing remote-editing problems come from being on the wrong machine.
- Some extensions show "Install in WSL: Ubuntu" rather than plain Install — because the extension needs to run where the code actually is.

### In practice
- Command Palette → connect command (`Remote-SSH: Connect to Host…` or `WSL: Connect to WSL`). New window opens; first connection installs a small server component remotely.
- Bottom-left indicator names the connected remote; Open Folder browses the REMOTE filesystem; integrated terminal = a shell on the remote machine.

### Boundary concept
- Interface is local; filesystem/language service/runtime may be remote. This local-interaction vs remote-execution split recurs later (containers, reproducible environments — Week 6).

## 3.16 Reading an Unfamiliar Codebase
- Workflow: open folder → inspect Explorer tree → look for entry points (main file, startup files, config dirs, scripts, top-level docs) → use file/text search for concepts/names from instructions or errors → use symbol-aware navigation (breadcrumbs, outline, go-to-definition) to trace deeper.

### Find the entry point first
- Don't just open a random file and hope — ask "where does execution probably begin?" (main function, web server startup, tests, route definitions, task-runner scripts) and trace outward from there.

### Concrete starting checklist
1. Skim top-level folder names in the Explorer.
2. Search for `main`, a filename matching the project name, or a term from a bug report.
3. Use go-to-definition + find-references to trace outward one hop at a time.
4. If a debugger is available, set a breakpoint at the entry point and step through rather than reading statically.

- Remote development boundaries (3.15) apply the same way here — same tools work locally or remotely.
- Editor proficiency (search, navigate, split, inspect, remote features) = directly tied to codebase comprehension speed and safety.

## 3.17 Cheat Sheet: Editor Shortcuts Worth Remembering

### Vim survival set
```
Open a file:         vim file.txt
Insert/Append/New:   i / a / o
Return to Normal:    Esc
Move:                h j k l | words: w b e | line: 0 $
Delete:              x  dd  dw
Copy/paste:          yy  p
Undo/Redo:           u  Ctrl+r
Search:              /pattern  n  N
Save/quit:           :w  :q  :wq  :q!
```
- To get properly comfortable: `vimtutor`.

### VS Code shortcuts (Windows/Linux · macOS)
| Shortcut | Action |
|---|---|
| Ctrl+P · Cmd+P | Quick Open — jump to file by name |
| Ctrl+Shift+P · Cmd+Shift+P | Command Palette — run any command by name |
| F12 (both) | Go to Definition (needs language support) |
| Ctrl+Shift+F · Cmd+Shift+F | Find in Files — search whole workspace |
| Ctrl+\ · Cmd+\ | Split Editor |
| Ctrl+Shift+L · Cmd+Shift+L | Select All Occurrences (multi-cursor) |
| F2 (both) | Rename Symbol — language-aware, updates all real references |
| F9 (both) | Toggle Breakpoint |

- Habit worth building: repeating the same 3-4 clicks → open Command Palette and search for what you're doing. There's very likely already a command (and often a shortcut).