# Week 6 Practice Test: Virtual Machines, Containers & Package Management

Attempt the questions before opening the separate solutions file. Explain your reasoning for multi-part questions, not only the command or label.

## Question 1: Runtime conditions

A Python program works on one laptop but fails on another. The source code is identical. Which statement is TRUE?

Select one:

A. Identical source guarantees identical behaviour if both users run the same command.

B. Only the Python package versions can explain the difference; operating-system and filesystem differences are irrelevant.

C. The language runtime, packages, system libraries, filesystem/configuration, and OS or hardware can all affect execution.

D. Creating a virtual environment automatically makes the entire operating system identical.

## Question 2: Dependencies and installed state

A project contains:

```text
requirements.txt
pyproject.toml
.venv/
src/report.py
```

### (a) [1.5 marks]

Distinguish between a dependency declaration, installed state, source code, and runtime evidence. Give one example from the listing or from a command output for each.

### (b) [1 mark]

Explain why `.venv/` is normally rebuilt rather than committed to Git and copied to another machine.

### (c) [1.5 marks]

A developer writes `requests>=2,<3`. Explain why this can produce a different installation later, even on the same project.

## Question 3: Choosing an isolation boundary

For each requirement, choose the most suitable first tool: a Python virtual environment, a container, or a virtual machine. Briefly justify each choice.

### (a) [1 mark]

Two Python projects need incompatible versions of the same Python library on one laptop.

### (b) [1 mark]

An application needs a Python package and a specific Linux command-line utility, but does not need a different kernel.

### (c) [1 mark]

You must run software that requires a complete legacy operating system and its kernel.

## Question 4: Docker build and run stages

Consider:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN python -m pip install -r requirements.txt
COPY src/ ./src/
CMD ["python", "src/main.py"]
```

### (a) [1.5 marks]

Identify which instructions run at build time and which command is the default run-time command.

### (b) [1 mark]

The developer edits `src/main.py` after building the image. Will an existing image automatically contain the edit? Explain.

### (c) [1.5 marks]

Why is copying `requirements.txt` and installing dependencies before copying frequently changing source useful for build caching?

### (d) [1 mark]

What is the difference between the Dockerfile, the image, and the container?

## Question 5: Reproducibility and package-manager boundaries

A teammate says: “I ran `pip install toolx`, so the application has everything it needs. If it fails on another computer, the other user installed it incorrectly.”

### (a) [2 marks]

Give two reasons this conclusion is too strong.

### (b) [1.5 marks]

Explain the difference between a pinned requirement such as `toolx==4.2.0` and a generated lockfile.

### (c) [1.5 marks]

The application also needs the OS command `ffmpeg`. Which package-manager scope normally provides it, and why can’t pip be assumed to provide it?
