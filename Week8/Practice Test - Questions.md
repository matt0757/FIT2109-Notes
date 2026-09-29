# Week 8 Practice Test: Debugging - Systematic Bug Hunting

Attempt the questions before opening the separate solutions file. Treat each scenario as an investigation: identify evidence before proposing a fix.

## Question 1: Debugging workflow

A user says an application “sometimes” produces the wrong total. Which action should happen first?

Select one:

A. Rewrite the function that calculates the total.

B. Add several unrelated defensive checks until the symptom disappears.

C. Record the exact inputs, environment, and steps that reproduce the problem.

D. Assume the database or library is responsible and upgrade it.

## Question 2: RIP model

Consider:

```python
def discount(price, code):
    if code == "STUDENT":
        price = price * 0.90   # should be 0.85
    return price
```

### (a) [1.5 marks]

For `discount(100, "STAFF")`, which RIP condition fails, and why is there no visible failure?

### (b) [1.5 marks]

For `discount(100, "STUDENT")`, explain how reachability, infection, and propagation all hold.

### (c) [1 mark]

Give one reason a bug might be present but not yet visible in a test suite.

## Question 3: Tracebacks and error evidence

A Python program prints:

```text
Traceback (most recent call last):
  File "app.py", line 30, in <module>
    result = process(records)
  File "app.py", line 14, in process
    return records[index]
IndexError: list index out of range
```

### (a) [1 mark]

What is the exception type and message?

### (b) [1 mark]

Which line should you inspect first, and why should traceback reading usually start from the bottom?

### (c) [1.5 marks]

Name two facts you would inspect before changing the code.

## Question 4: Logging and static analysis

### (a) [1.5 marks]

Order these Python logging levels from least severe to most severe: `ERROR`, `DEBUG`, `CRITICAL`, `INFO`, `WARNING`.

### (b) [1 mark]

If logging is configured at `WARNING`, which of those levels are normally shown?

### (c) [1.5 marks]

What can static analysis detect that runtime testing may miss, and what can runtime testing detect that static analysis cannot prove by itself?

## Question 5: Native debugging

A C program terminates with:

```text
Segmentation fault (core dumped)
```

### (a) [1 mark]

Why is this message alone insufficient to identify the faulty source line?

### (b) [1.5 marks]

Give a suitable compile command and the first debugger command you would use after reproducing the crash.

### (c) [1.5 marks]

Explain the difference between `next`, `step`, and `backtrace` in GDB or LLDB.

## Question 6: Minimal reproducible examples

A 400-line data pipeline fails only for one input file. You want to report the bug and investigate it efficiently.

### (a) [2 marks]

Describe how to reduce it to a minimal reproducible example while preserving the failure.

### (b) [1 mark]

Why should you remove or simplify one part at a time and rerun the reproducer after each removal?

### (c) [1 mark]

How does `git bisect` apply a related binary-search idea to regressions?
