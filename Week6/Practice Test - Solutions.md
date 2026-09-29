# Week 6 Practice Test: Solutions and Learning Notes

## Question 1: Runtime conditions

**Answer: C.**

“Same code” does not mean “same execution conditions.” Differences can exist in the Python version, package releases, system libraries, paths, permissions, environment variables, operating system, or hardware. A virtual environment isolates Python packages, but it does not recreate the whole machine.

**Common trap:** treating a successful install or a passing test on one machine as proof that every layer is compatible.

**Remember:** repeatability means you can reconstruct a setup; compatibility still needs to be tested.

## Question 2: Dependencies and installed state

### (a)

- **Dependency declaration:** `requirements.txt` or `pyproject.toml`; it records constraints or project metadata.
- **Installed state:** `.venv/` and its `site-packages/`; it is the generated environment currently installed on this machine.
- **Source code:** `src/report.py`; it is the project’s own implementation.
- **Runtime evidence:** output such as `python -c "import sys; print(sys.executable)"`, a package version, or a package path. It shows what actually ran.

The distinction matters because a declaration describes intent, while runtime evidence verifies the actual interpreter and packages used.

### (b)

A virtual environment contains platform-specific files and interpreter references. It is not reliably portable between operating systems, machines, or even different Python installations. The usual practice is to commit the dependency description, exclude `.venv/`, and rebuild it with the intended interpreter.

### (c)

`requests>=2,<3` permits any compatible version from the 2.x range. A newer release, changed transitive dependency, withdrawn release, Python version, platform, or package index state can cause the resolver to choose a different set later.

**Common trap:** a version range improves compatibility options but does not make the complete dependency graph identical.

**Remember:** requirements describe allowed choices; runtime checks show the choices actually installed.

## Question 3: Choosing an isolation boundary

### (a)

Use a **Python virtual environment**. The conflict is only in Python package locations, so separate `site-packages/` directories are sufficient.

### (b)

Use a **container**. The application needs both Python packages and a system utility, so a container can capture the user-space filesystem, system packages, files, and command. It still shares the host kernel.

### (c)

Use a **virtual machine**. A VM provides a complete guest operating system and guest kernel on virtualised hardware.

**Common trap:** choosing the strongest tool automatically. The right boundary is determined by where the dependency lives: Python package, user-space system files, or a complete OS/kernel.

## Question 4: Docker build and run stages

### (a)

`FROM`, `WORKDIR`, both `COPY` instructions, and `RUN` are processed while building the image. `CMD ["python", "src/main.py"]` records the default command used when a container is started from that image.

### (b)

No. The build copied the source as it existed at build time. An existing image is not updated when the host file changes. The image must be rebuilt, unless the developer deliberately uses a runtime mount that supplies the changed file.

### (c)

If dependency files and the install instruction have not changed, the builder can reuse that earlier layer. Source edits then invalidate only later layers, so dependencies do not need to be installed again for every small source change.

Caching improves speed; it does not silently update an image or replace verification.

### (d)

- **Dockerfile:** the reviewable build recipe.
- **Image:** the constructed, reusable user-space contents and configuration.
- **Container:** a running instance created from the image.

**Remember:** Dockerfile -> image -> container is recipe -> artifact -> runtime instance.

## Question 5: Reproducibility and package-manager boundaries

### (a)

The conclusion is too strong because:

1. `pip` manages Python packages, not necessarily OS commands, shared libraries, fonts, paths, permissions, or remote services.
2. Dependency resolution can choose different transitive versions or platform-specific builds. The other machine may also use a different Python version or operating system.

A successful install proves only that one package-manager operation succeeded in one environment.

### (b)

`toolx==4.2.0` is a human-maintained exact requirement for that named package. It improves repeatability for that package but does not necessarily record every transitive dependency or the full platform context. A generated lockfile records the tool’s complete resolved dependency graph and related metadata for repeatable reconstruction.

### (c)

`ffmpeg` is normally an OS-level dependency supplied by a system package manager such as `apt` on Ubuntu/Debian, or by the corresponding manager on another OS. Pip installs Python packages into a Python environment; it does not generally install arbitrary system commands or configure the OS.

**Remember:** identify the scope first: pip for Python packages, npm for Node project packages, apt for OS packages, and so on.
