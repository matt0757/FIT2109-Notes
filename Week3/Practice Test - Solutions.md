# Week 3 Practice Test: Solutions and Learning Notes

## Question 1: Files and buffers

**Answer: B.**

A file is the persistent object on disk; a buffer is the editor's in-memory state. They can differ while edits are unsaved. Closing without saving normally loses the unsaved buffer changes, while the last saved file remains on disk. VS Code's hot exit is a deliberate exception that can preserve unsaved state across sessions.

**Common trap:** treating a tab as a separate file. A tab is a visible representation of an open editor item.

**Remember:** save is the point where buffer state becomes file state.

## Question 2: Vim modes and commands

### (a)

Vim opens in **Normal mode**. Press `i` to enter Insert mode. Press `Esc` to return to Normal mode.

### (b)

- `dd` deletes the current line.
- `yy` yanks, or copies, the current line.
- `p` pastes after the cursor.

### (c)

`:wq` saves and quits. `:q!` quits without saving.

The colon enters command-line mode from Normal mode.

## Question 3: Navigation tools

### (a)

**Quick Open** (`Ctrl+P`) fuzzy-matches filenames.

### (b)

**Search** (`Ctrl+Shift+F`) searches across the project or workspace.

### (c)

**Go to Definition** (`F12`) uses language-aware information to navigate from the symbol at the cursor to its definition.

**Remember:** Find is current-file text search; Search is project-wide text search; semantic navigation understands code structure.

## Question 4: Formatting, linting, and language support

### (a)

A formatter answers “how should this code look?” by applying layout and style rules. A linter answers “does anything look suspicious?” by reporting issues such as unused imports, unreachable code, or risky patterns.

### (b)

Check whether VS Code can find the intended Python interpreter. Completion, diagnostics, and formatting often depend on several extensions and the selected interpreter; a broken or missing interpreter can make all of them appear to fail together.

### (c)

Rename Symbol uses language-aware references and can distinguish actual symbol uses from comments, strings, and similarly named text. A text replace-all operation cannot make that distinction.

## Question 5: Editor workflow scenario

### (a)

Use two **editor groups**, or panes, to show the files side by side.

### (b)

Use multiple selections or multi-cursor editing to select and change repeated occurrences together.

### (c)

A workspace-oriented editor exposes the project tree, open files, search, language symbols, diagnostics, terminals, version-control integration, and related files in one context. This reduces the cost of tracing a change across files and helps catch errors through tooling while editing.

**Remember:** editor fluency is not just typing speed; it is navigating, inspecting, changing, and verifying with the right tool.
