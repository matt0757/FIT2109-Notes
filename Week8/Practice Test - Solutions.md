# Week 8 Practice Test: Solutions and Learning Notes

## Question 1: Debugging workflow

**Answer: C.**

First capture the exact inputs, environment, and ordered steps that trigger the problem. A reliable reproduction gives you something to measure and lets you verify whether a later change actually fixed the failure.

**Common trap:** editing the most suspicious code immediately or making several changes at once. That destroys evidence and makes the result impossible to interpret.

**Remember:** observe -> hypothesise -> change one thing -> compare the result.

## Question 2: RIP model

### (a)

**Reachability fails.** The faulty multiplication line runs only when `code == "STUDENT"`. With `"STAFF"`, the line is never executed, so the incorrect calculation cannot infect the state or propagate to the output.

### (b)

- **Reachability:** the condition is true, so the faulty line executes.
- **Infection:** the price becomes `90` instead of the correct `85`.
- **Propagation:** that incorrect price is returned to the caller, so the visible output is wrong.

### (c)

The tests may never exercise the faulty branch or input category, so reachability never holds. Alternatively, the wrong state might be overwritten before the output, so propagation fails. A test suite can pass without proving that every path and output condition has been exercised.

**Remember:** a bug becomes visible only when R, I, and P all hold.

## Question 3: Tracebacks and error evidence

### (a)

The exception is `IndexError`, with the message `list index out of range`.

### (b)

Inspect `app.py` line 14 first because that is where the exception was raised in the program's own code. The bottom of the traceback gives the concrete error type and message, while the nearest relevant frame shows the operation that failed. The upper frames mainly explain how execution reached that point.

### (c)

Inspect facts such as:

- the value of `index`;
- `len(records)`;
- the input that produced the failure;
- the caller's assumptions about whether records can be empty;
- the current working directory or environment if data loading may be involved.

The point is to inspect actual state rather than assume that the index or input has the expected value.

## Question 4: Logging and static analysis

### (a)

`DEBUG` -> `INFO` -> `WARNING` -> `ERROR` -> `CRITICAL`.

### (b)

`WARNING`, `ERROR`, and `CRITICAL` are normally shown. A configured level acts as a threshold: that level and more severe levels pass through.

### (c)

Static analysis examines source without running it. It can find unused imports, undefined names, type mismatches, unreachable code, suspicious patterns, and style problems. Runtime testing observes actual behavior for particular inputs and environments, so it can reveal wrong outputs, crashes, timing issues, and environment-dependent failures that static analysis cannot prove from source alone.

**Common trap:** treating a clean linter result as proof that the program is correct. Static analysis and runtime evidence answer different questions.

## Question 5: Native debugging

### (a)

A segmentation fault reports an invalid memory access, but the memory may have been corrupted earlier than the line where the process finally crashes. The message does not include enough source or call-stack information by itself.

### (b)

Compile with debug symbols, for example:

```bash
gcc -g -o crash crash.c
```

After reproducing the crash under GDB or LLDB, use `backtrace` in GDB or `bt` in LLDB first. This shows the call stack and identifies the active functions and source locations.

### (c)

- `next` steps over a function call without entering it.
- `step` enters a function call so its execution can be inspected.
- `backtrace` or `bt` prints the current call stack, showing how execution reached the current frame.

**Remember:** compile for evidence (`-g`), reproduce, then inspect the stack before guessing.

## Question 6: Minimal reproducible examples

### (a)

Copy the failing code into a fresh file, then remove unrelated imports, helpers, output, and processing stages. Replace file, network, or database inputs with the smallest hardcoded data that still triggers the bug. After each reduction, run the same test command and retain only changes that preserve the failure. The final example should be the smallest syntactically valid program and input that still demonstrates the problem.

### (b)

Removing several parts at once can remove the cause or change the behavior without revealing which part mattered. One-at-a-time reduction gives a clear experiment: if the bug remains, the removed part was not necessary; if it disappears, restore that part and investigate it further.

### (c)

`git bisect` tests commits between a known-good and known-bad revision, repeatedly selecting a midpoint. Mark each tested commit good or bad using the same test until Git identifies the first commit that introduced the regression.

**Remember:** an MRE searches the program structure; `git bisect` searches the commit history. Both use controlled binary reduction.
