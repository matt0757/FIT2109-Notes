# Week 8: Debugging - Systematic Bug Hunting

## 8.1 Debugging Is Disciplined Hypothesis Testing
- Debugging = scientific method applied to bugs, not random guessing.
- Cycle: **Observe** (exact error, inputs, environment) → **Hypothesise** (2-3 candidate causes) → **Experiment** (change ONE thing) → **Conclude** (did symptom change as predicted?).
- **Common trap**: changing 2+ things at once = not an experiment. You won't know what fixed it (or why it didn't).

### Five-step debugging workflow
| Step | What you do | Why |
|---|---|---|
| Reproduce | Write exact steps/inputs/env that trigger it reliably | Can't fix reliably what you can't reproduce |
| Isolate | Narrow to specific function/line/condition | Small target = fast fix |
| Inspect | Use logs/debugger/prints to see actual state | Assumptions are often wrong |
| Fix | Minimal change addressing root cause | Symptom fixes resurface later |
| Verify | Re-run repro steps + related tests | Confirms fix, checks no regressions |

### Common traps
- Assuming the bug is in someone else's code (usually isn't — check yours first).
- Fixing the **symptom**, not the cause (e.g. adding a None check instead of asking why it's None).
- **Debugging by addition** — adding code before understanding, makes things harder to reason about.
- Trusting memory over evidence — verify with a breakpoint/log, don't assume.

## 8.2 The RIP Model: From Fault to Failure
- A bug in code doesn't always cause visible failure — 3 conditions must ALL hold:

| Letter | Condition | Meaning |
|---|---|---|
| **R** | Reachability | Faulty line must actually execute |
| **I** | Infection | Executing it must corrupt program state |
| **P** | Propagation | Corrupted state must reach the output |

- If any ONE fails → bug exists but is invisible (until some future input triggers all three).

### Example
```python
def discount(price, code):
    if code == "STUDENT":
        price = price * 0.9   # bug: should be 0.85
    return price

discount(100, "STAFF")    # R fails — line never reached — no visible bug
discount(100, "STUDENT")  # R,I,P all hold — visible wrong output
```

### Why each can fail independently
| Condition | Why it fails | Symptom |
|---|---|---|
| R | Faulty code in untested branch/path | "works on my test cases" |
| I | Line runs but happens to still give correct result | Bug appears intermittently |
| P | Wrong value gets overwritten/corrected downstream | "wrong at line 12 but output is fine" |

### RIP explains common frustrations
- Bug disappears when you add a print — the print itself can alter timing/memory enough to suppress R or I. Bug is still there.
- Works locally, fails in CI — different inputs/env change whether R or I holds.
- Tests pass, production crashes — tests never hit R (untested path).
- Fixed the crash, output still wrong — you patched where P collapsed, not the original fault.

### How to use RIP debugging (work backwards)
1. Start at the failure (P collapsing) — what wrong output did you see?
2. Trace back the infected state (I) — which variable became wrong, and where?
3. Find the fault (R) — which line caused it, and confirm it's actually executed for this input?

## 8.3 Reading Error Messages and Tracebacks
- Reading the error carefully = highest-leverage debugging action. Don't skip to changing code.

### Python tracebacks — read bottom to top
```
Traceback (most recent call last):
  File "app.py", line 18, in <module>
    result = process(data)
  File "app.py", line 9, in process
    return data[index]
IndexError: list index out of range
```
- **Bottom line** = error type + message (what happened).
- **Line above** = exact file/line where it was raised (where to look first).
- **Stack above that** = call chain — scan up for the last line of YOUR code (not library code) = likely root cause.
- Mistake: reading top-down leads you to outer/library code you can't fix.

### Common Python error types
| Error | Meaning | Check first |
|---|---|---|
| SyntaxError | can't parse file | line shown (often line BEFORE the typo) |
| NameError | name not defined | spelling, scope, init order |
| TypeError | wrong type for operation | `type(x)` |
| IndexError | list index doesn't exist | list length vs index |
| KeyError | dict key doesn't exist | check with `.get()` |
| AttributeError | object lacks that attribute/method | might be `None` or wrong type |
| FileNotFoundError | path doesn't exist | exact path, relative vs absolute, cwd |

### Java exceptions
- Same bottom-up logic: first line = exception class + message; stack frames listed innermost-first (top-most frame in YOUR code = likely root cause).
```
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 3
    at com.example.App.getItem(App.java:12)
    at com.example.App.process(App.java:7)
    at com.example.App.main(App.java:3)
```
- Distinguishes **checked** exceptions (compiler forces handling) vs **unchecked** (runtime surprises).

| Exception | Meaning |
|---|---|
| NullPointerException | accessed method/field on null (most common) |
| ArrayIndexOutOfBoundsException | invalid array index |
| ClassCastException | incompatible type cast |
| StackOverflowError | infinite/unbounded recursion |
| NumberFormatException | string can't parse as number |

### C runtime errors
- C has no exceptions — errors show as crashes/signals, and are often far from the actual bug.
```
Segmentation fault (core dumped)    # accessed memory you don't own
Abort trap: 6                        # assert() failed or heap corruption
Floating point exception             # e.g. divide by zero
```
- **Segfault** = tried to access memory not allowed (NULL deref, buffer overrun, use-after-free). Message alone gives no location — need a debugger (GDB, compile with `-g`).
- C errors are silent until they crash — corruption can happen lines/functions before the actual crash. **AddressSanitizer** (`-fsanitize=address`) catches errors at the point they occur.

### Shell command errors
- Use exit codes, not exceptions. `0` = success, non-zero = failure.
```bash
cat nonexistent.txt
echo $?    # check exit code right after
```
- Read the full error line (usually names the command + short reason) before searching online.

## 8.4 Log Levels — Separating Noise from Emergencies
- Log levels let you filter messages by severity.

| Level | Value | Use for | Shown by default? |
|---|---|---|---|
| DEBUG | 10 | Fine detail for investigation | No |
| INFO | 20 | Normal milestones | No |
| WARNING | 30 | Unexpected but recoverable | Yes |
| ERROR | 40 | Recoverable failure | Yes |
| CRITICAL | 50 | Fatal, can't continue | Yes |

- Setting a level = filter: shows that level and above.

### Python logging setup
```python
import logging
logging.basicConfig(level=logging.DEBUG, format='%(levelname)-8s %(name)s: %(message)s')
logger = logging.getLogger(__name__)

logger.debug("loop iteration %d, value=%r", i, val)
logger.info("processing complete: %d items", count)
logger.warning("retrying after timeout")
logger.error("failed to write output: %s", e)
logger.critical("database unreachable, shutting down")
```
- Why not `print()`? A `logger.debug()` costs almost nothing when level is set higher (message never formatted). No code changes needed to toggle verbosity.
- Rule of thumb: production → INFO/WARNING; debugging → DEBUG; libraries → always `getLogger(__name__)`, never `basicConfig` (let the app decide).

### Java (SLF4J/Logback)
```java
private static final Logger logger = LoggerFactory.getLogger(MyClass.class);
logger.debug("loop iteration {}, value={}", i, val);
logger.warn("retrying after timeout");
logger.error("failed to write output", e);
```

### C
- No built-in framework — write to `stderr` with a severity prefix.
```c
fprintf(stderr, "[DEBUG] loop iteration %d, value=%d\n", i, val);
fprintf(stderr, "[ERROR] failed to open file: %s\n", path);
/* or syslog for daemons */
```

## 8.5 Static Analysis: Catching Bugs Before Runtime
- Examines source code WITHOUT running it — finds unused vars, type mismatches, style issues, suspicious patterns.
- Cheaper to fix a bug caught by a linter than one found in production.

### What it finds
- Unused imports/variables, vars used before defined, type mismatches, unreachable code after return, style violations (PEP 8), risky patterns (`eval()`, shell injection).

### Tools by ecosystem
| Tool | Language | Purpose |
|---|---|---|
| pylint | Python | Full linter, style+logic+score/10 |
| flake8 | Python | Lightweight PEP 8 checker |
| mypy | Python | Static type checker |
| Checkstyle | Java | Style enforcement |
| SpotBugs | Java | Real bugs (null deref, leaks) via bytecode |
| clang-tidy | C/C++ | Memory errors, UB, style |
| shellcheck | Bash/sh | Quoting mistakes, undefined vars |
| ESLint | JS/TS | Unused vars, type issues, style |

### Reading pylint output
```
bad_script.py:4:0: W0611 (unused-import) Unused import os
bad_script.py:9:8: E0602 (undefined-variable) Undefined variable 'resut'
```
- Format: `file:line:col: CODE (id) Message`. Prefix: **E**=error, **W**=warning, **C**=convention, **R**=refactor. Fix E first.

### shellcheck example
```
In bad_script.sh line 5:
  for f in $(ls *.txt); do
                ^-- SC2045: Iterating over ls output is fragile.
```
- Best practice: run linters in CI on every push, fail build on errors.

## 8.6 Reproduce Before You Fix
- **Golden rule**: never attempt a fix until you can trigger the bug on demand.

### To reproduce, capture
1. **Exact inputs** (data, files, actions)
2. **Environment** (OS, versions, env vars, cwd)
3. **Steps** (precise sequence, in order)
- Writing it down like a bug report often reveals the cause itself.
- **Intermittent bugs** → suspect timing/ordering/concurrency (race conditions, filesystem delays, random seeds). Add logging, run many times.

### Isolating with binary search
- Same principle as searching a sorted list — divide the pipeline in half repeatedly.
```python
def process(data):
    step1 = transform(data)
    print("after transform:", step1)   # correct?
    step2 = filter_records(step1)
    print("after filter:", step2)      # still correct?
    return aggregate(step2)
```
- `git bisect` = automates the same binary search over commit history (see Week 5, 5.6).

### Reproduce checklist
| Question | Rules out |
|---|---|
| Reproduces on a clean run? | leftover state from prior run |
| Same inputs → same result every time? | randomness/external data changes |
| Reproduces on another machine/container? | machine-specific environment |
| Reproduces on current main? | already-fixed bug |

## 8.7 GDB and LLDB: Inspecting Native Programs
- For C/C++ crashes ("Segmentation fault" alone tells you nothing) — a debugger lets you pause and inspect state/call stack.
- **GDB** = standard on Linux. **LLDB** = default on macOS (LLVM), also runs on Linux. Nearly identical commands.

### Step 0: compile with debug symbols
```bash
gcc -g -o crash crash.c   # -g embeds line numbers/variable names
```
- Don't combine `-g` with `-O2`+ (optimisation reorders/removes code, confusing line numbers). Use `-g` alone or with `-O0`.
- Note: `-g` itself can occasionally alter code generation and expose latent compiler bugs (rare, but documented in compiler research) — if a crash disappears when removing `-g`, it might be a compiler issue.

### Essential commands
| GDB | LLDB | Does |
|---|---|---|
| `gdb ./crash` | `lldb ./crash` | start debugger |
| `break main` | `b main` | breakpoint at main |
| `break crash.c:12` | `b crash.c:12` | breakpoint at specific line |
| `run` | `run` | start/restart program |
| `next` | `next` | step over (don't enter functions) |
| `step` | `step` | step into functions |
| `continue` | `continue` | resume to next breakpoint/crash |
| `print total` | `p total` | print variable value |
| `backtrace` | `bt` | show full call stack |
| `frame 1` | `frame select 1` | move to a call-stack frame |
| `list` | `list` | show source around current line |
| `quit` | `quit` | exit |

### Typical crash session
```
$ gdb ./crash
(gdb) run
Program received signal SIGSEGV, Segmentation fault.
0x000000000040115a in fill_array (arr=0x0, n=10) at crash.c:8
(gdb) backtrace
#0  fill_array (arr=0x0, n=10) at crash.c:8
#1  main () at crash.c:17
(gdb) frame 1
(gdb) print arr
$1 = (int *) 0x0
```
- **First command after a crash**: always `backtrace` — most useful single piece of info.

## 8.8 VS Code Debugger
- GUI version of GDB-style concepts (breakpoints, stepping, variables, call stack) — works for Python, JS, Go, etc.

### Key panels
- **Breakpoint** (red dot in gutter) — click line number to set/remove. Right-click for conditional breakpoints.
- **Variables panel** — all local/global vars in scope at the pause point.
- **Watch panel** — evaluate custom expressions live at every step (e.g. `len(results)`, `total/count`).
- **Call Stack panel** — click any frame to inspect its locals.
- **Debug Console** — REPL in context of the paused program.

### Keyboard shortcuts
| Key | Action | Use |
|---|---|---|
| F5 | Continue | run to next breakpoint |
| F10 | Step Over | run function without entering |
| F11 | Step Into | enter the function call |
| Shift+F11 | Step Out | finish current function, return to caller |
| Shift+F5 | Stop | kill debug session |

### launch.json (minimal Python config)
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug current file",
      "type": "debugpy",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal"
    }
  ]
}
```
- Press F5 to launch.

### Watch panel — deeper use
- Add expressions (not just variable names): `len(results)`, `total/count`, `items[i]`, `expected == actual`, `type(data)`.
- Watch = track specific/derived values over many steps. Variables = broad picture at one pause point.
- Watch expressions persist across sessions. Can call functions (careful with side effects!).
- Better than `print()`: can add/remove/change tracked values without touching source or re-running.

### Data breakpoints (watchpoints)
- Pause automatically when a variable's value changes (no manual stepping).
- In VS Code: pause once → right-click variable in Variables panel → "Break on Value Change" → F5.
- Support varies by language/extension (reliable for C/C++, may be limited for Python/debugpy — use conditional breakpoints instead if greyed out).

### Conditional breakpoints (universal alternative)
- Right-click breakpoint → "Edit Breakpoint" → choose "Expression" → type condition (e.g. `total < 0`, `i == 999`, `result is None`).
- Pauses only when the condition is True. Dot gains a `=` symbol.

### GDB/LLDB native watchpoints
```
(gdb) watch total       # pause on any write
(gdb) rwatch total      # pause on any read
(gdb) awatch total      # pause on read or write
(gdb) continue
```

## 8.9 Minimal Reproducible Examples (MRE)
- **MRE** (aka MWE/MCVE) = smallest program that still triggers the bug — every non-essential line removed.

### Why bother
- Faster to reason about (small = fits in your head).
- Reveals the true cause — irrelevant code hides bugs.
- Required for good bug reports (Stack Overflow, GitHub Issues expect one).
- Making it often *is* the fix — simplifying reveals the wrong assumption.

### Example
```python
# MRE — just 4 lines
data = [1, 2, 3]
index = len(data)        # off-by-one
print(data[index])       # IndexError
```
(vs a 200-line pipeline where none of the rest mattered)

### How to create one
1. Copy failing code into a fresh empty file.
2. Remove unneeded imports — re-run after each removal to confirm bug persists.
3. Delete unrelated helper functions/classes.
4. Replace file I/O / network / DB calls with hardcoded literal data.
5. Simplify inputs to smallest values that still trigger it.
6. Repeat until removing ANY further line stops the bug.
- **Golden rule**: run and confirm the bug still appears after EVERY removal — don't remove too much at once.

### Rubber duck debugging
- Explaining the bug (to a person or an imaginary duck) forces precise articulation — often reveals the cause. MRE-building has the same effect.

### MREs in Java/C
- Java: single class + single `main`, hardcoded literals, compile/run with `javac Repro.java && java Repro`.
- C: copy into new `.c` file, remove unrelated functions, hardcode arrays, confirm with `gcc -g -o repro repro.c && ./repro` — segfault MREs are often under 10 lines.

## 8.10 Perses: Automated Program Reduction
- Automates MRE-creation for large files: give it the buggy program + a test script, it removes code while keeping the bug present.
- Uses the language's **grammar** to remove only syntactically valid units (statements/expressions/function bodies) — unlike naive line-deletion tools (e.g. creduce), so it never produces broken syntax.

### Algorithm (delta debugging variant)
1. Start with original buggy program.
2. Try removing a chunk; run test script.
3. Bug still present (exit 0) → keep removal, continue.
4. Bug gone (exit non-zero) → restore chunk, try different one.
5. Repeat until no further removal preserves the bug.

### Setup
```bash
java --version    # requires Java 11+
# install if missing: brew install openjdk / sudo apt install default-jdk

curl -L -o perses_deploy.jar \
  https://github.com/uw-pluverse/perses/releases/latest/download/perses_deploy.jar
java -jar perses_deploy.jar --help
```

### Test script rules (critical)
- **Exit code convention is REVERSED from normal shell**: exit `0` = bug STILL present (keep reducing); non-zero = bug gone (stop this path).
- **Do NOT use `$1`** — Perses passes no argument; reference the file by its literal name.
```bash
#!/bin/bash
# test.sh — exit 0 if big_bug.py still triggers IndexError
python3 big_bug.py 2>&1 | grep -q "IndexError"
```
```bash
chmod +x test.sh
bash test.sh && echo "bug confirmed"   # verify BEFORE running Perses
```

### Running Perses
```bash
java -jar perses_deploy.jar \
    --test-script "$(pwd)/test.sh" \
    --input-file "$(pwd)/big_bug.py"
```
- Use **absolute paths** (`$(pwd)/...`) — Perses changes directories internally, relative paths can silently fail.

### Useful flags
| Flag | Does |
|---|---|
| `--input-file FILE` | buggy file to reduce |
| `--test-script SCRIPT` | test script (must be executable) |
| `--output-dir DIR` | output location (default `perses_result/`) |
| `--threads N` | parallel test runs |
| `--keep-original-code-format` | preserve whitespace/formatting |

- Supports Python, C, C++, Java, Go, Rust, JavaScript, and more (auto-detects grammar by file extension).
- Use Perses for files >~50 lines or for formal bug reports; manual reduction is faster for short files.

### Troubleshooting "initial sanity check failed"
- Means the test script returned non-zero every time (bug not detected) — checklist:
  1. Using `$1` in test script (most common mistake) — fix: reference filename directly.
  2. Use absolute paths (`$(pwd)/...`) for both flags.
  3. Verify test script manually first (`bash test.sh && echo "bug confirmed"`).
  4. CRLF line endings (Windows/WSL) — fix with `dos2unix test.sh` or `sed -i 's/\r//' test.sh`.
  5. `python3` vs `python` command mismatch — check with `python3 --version`.
  6. Test script not executable — `chmod +x test.sh`.
  7. Wrong error string in `grep` — confirm actual output matches the pattern searched.

## 8.11 "Works on My Machine" — Environment Differences
- A bug that only appears in one environment isn't in the code — it's in the environment.

### Common root causes
| Cause | Example | Detect with |
|---|---|---|
| Runtime version | f-strings need Python 3.6+ | `python3 --version` on both machines |
| Library version mismatch | Local pinned version vs CI's latest | compare `pip freeze` outputs |
| Missing env variable | `os.environ["API_KEY"]` set locally, absent in CI | `printenv \| grep API`; check CI secrets |
| Path differences | Relative path resolves differently by cwd | print `os.getcwd()` and abspath at startup |
| OS behavior | `\` vs `/` in file paths | run on target OS/container; use `os.path.join` |
| File permissions | restricted user in container | `ls -la`, `whoami` |
| Locale/encoding | UTF-8 file, ASCII-locale server crashes | `locale`; open with `encoding="utf-8"` explicitly |

### Diagnosing environment bugs
1. Collect facts from both environments (runtime version, packages, env vars, cwd).
2. Compare — look for differences.
3. Reproduce the failing environment locally (container, venv, or inspectable CI job).
```bash
python3 --version
pip freeze > requirements_snapshot.txt
printenv | sort
python3 -c "import sys; print(sys.path)"
```

### Prevention strategies
- Pin dependencies (use a lockfile).
- Use environment isolation (per-project venvs etc).
- Use containers for CI — matches production, eliminates OS/library differences.
- Never hardcode absolute paths — use relative paths anchored to source file location.
- Validate required env vars at startup, fail early with a clear message.

### Dependency management by language
| Language | Dependency file | Isolation tool | Snapshot command |
|---|---|---|---|
| Python | requirements.txt | `python3 -m venv .venv` | `pip freeze > requirements.txt` |
| Java (Maven) | pom.xml | Maven Wrapper (`./mvnw`) | exact `<dependency>` versions, avoid ranges |
| JavaScript | package.json + package-lock.json | node_modules/ (auto) | `npm ci` |
| C/C++ | CMakeLists.txt/Makefile (no standard) | containers most reliable | document versions / conan.lock / vcpkg.json |

- Containers solve this by bundling OS + runtime + libraries + code into one image — identical everywhere.

---

## Week 8 Cheatsheet: Tools at a Glance

| Situation | Reach for | Quick command |
|---|---|---|
| Why a bug isn't visible | RIP model | Is line reached? Does it corrupt state? Does it reach output? |
| Catch issues before running | Lint (pylint/flake8/shellcheck) | `pylint myscript.py` |
| Crash with error message | Read traceback bottom-up | Find last line of your own code |
| Need more runtime detail | Logging | `logging.basicConfig(level=logging.DEBUG)` |
| C/C++ crash on Linux | GDB | `gdb ./program` → `run` → `backtrace` |
| C/C++ crash on macOS/LLVM | LLDB | `lldb ./program` → `run` → `bt` |
| Python/JS logic bug | VS Code debugger | gutter click → F5 → F10 to step |
| Need to file a bug report | MRE | strip to <20 lines still triggering bug |
| Large file, MRE tedious | Perses | `java -jar perses_deploy.jar --test-script test.sh --input-file bug.py` |
| Bug only in CI/another machine | Environment diff | `python3 --version`, `pip freeze`, `printenv` |

### Tool quick reference
- **pylint** — Python linter, score/10. `pip install pylint` → `pylint script.py` (fix E codes first).
- **flake8** — lightweight PEP 8 checker. `pip install flake8` → `flake8 script.py`.
- **mypy** — Python static type checker. `pip install mypy` → `mypy script.py` (needs type annotations).
- **shellcheck** — Bash/sh linter. `shellcheck script.sh`.
- **GDB** — C/C++ debugger, Linux. Compile with `-g`. Key: `break`, `run`, `next`, `step`, `print`, `backtrace`.
- **LLDB** — same workflow, default on macOS (`p`, `bt` instead of `print`, `backtrace`).
- **Perses** — automated program reducer (needs Java). Test script exit 0 = bug present.
- **git bisect** — binary search over commit history to find the bad commit (see Week 5).