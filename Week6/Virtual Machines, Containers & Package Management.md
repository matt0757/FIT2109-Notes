# Week 6: Virtual Machines, Containers & Package Management

## 6.1 The Runtime Environment
- "Same code" ≠ "same execution conditions." Code runs through surrounding layers that can differ.
- Layers that can break a program:

| Layer | Example difference |
|---|---|
| Language runtime | Python 3.9 vs 3.12 (different syntax/behavior) |
| Packages | Different releases expose different functions/defaults |
| System tools/libraries | Missing command-line tool or shared library |
| Filesystem/config | Different paths, permissions, env vars |
| OS/hardware | OS version, processor type affects compiled software |

- Why control matters: **debugging** (need to recreate exact env), **CI** (passing test = evidence about that env), **team handoff** (rebuild from docs, not copy machines), **deployment** (match validated setup).
- **Repeatability ≠ compatibility**: pinning a version = repeatable, but only testing proves it *works*.
- Containers reduce variation but don't remove every difference (OS behavior, hardware, network, secrets still vary).

## 6.2 Packages, Dependencies & Resolution
- One package install can pull in many others — and results can change over time even with the same command.
- **Package** = distributable software unit (name, version, metadata, own dependencies).
- **Package index** = online source of packages (e.g. PyPI for Python).
- **Dependency resolution** = installer (e.g. pip) picks a compatible set of versions based on constraints, Python version, platform.

### Direct vs transitive dependencies
- **Direct** = named by your app directly.
- **Transitive** = dependency of your dependency (you never import it, but still need a compatible version).

### Constraint styles
| Declaration | Meaning | Effect |
|---|---|---|
| `reportlib` | no version constraint | may get a newer release later |
| `reportlib==3.2.1` | exact version | repeatable while it's available |
| `reportlib>=3,<4` | range | resolver may pick different versions over time |

- Neither pinning nor ranges *prove* correctness — only tests do.
- Resolution changes because: new releases appear, old ones get withdrawn, transitive deps change, some releases only support certain platforms/Python versions.

### Requirements files vs lockfiles
- `requirements.txt` = declared constraints (may be exact, range, or mixed) — NOT guaranteed to fix every transitive version.
- **Lockfile** (tool-generated) = records the full resolved set (often with integrity/platform info).
- Declaration = "what may be installed." Lockfile = "what WAS installed, exactly."
- Package declarations do NOT choose OS, provide system commands, or isolate projects (later layers handle that).

## 6.3 Description vs Installed State
- Four kinds of project info:

| Category | Example | Role |
|---|---|---|
| Source | `app.py` | project's own code |
| Dependency description | `requirements.txt`/`pyproject.toml` | constraints, reviewed |
| Installed state | `.venv/`, `site-packages/` | generated from description |
| Runtime evidence | interpreter path, package path, version output | shows what actually ran |

- Source + dependency descriptions → version control. Installed state → rebuilt per machine (not committed).

### venv (virtual environment)
- Creates a **project-specific** Python package location (`site-packages/`), separate from other projects.
- Inherits the Python **version** of the interpreter used to create it (doesn't download a new one).
- **Activation** = adds venv's launcher to front of `PATH` for current shell only.
  - Does NOT copy the environment or reconfigure the machine permanently.
  - New terminal = need to activate again.
- Running `pip` through a specific interpreter determines exactly which environment gets the package (check both python path AND package path, not just the prompt).
- **venvs are not portable** — contain platform-specific files/interpreter references. Don't copy between machines; rebuild from the description instead (`.venv/` excluded from Git).
- venv boundary: only isolates Python packages — does NOT give system commands, a separate filesystem, or a new OS kernel.

## 6.4 Package Managers Have Boundaries
- No universal package manager — pip, npm, apt, Homebrew each manage a different **scope**.

| Scope | Manager | Manages |
|---|---|---|
| Python env | pip | Python packages for one interpreter |
| JS project | npm | Node.js project packages |
| Ubuntu/Debian | apt | OS-level commands/libraries/services |
| macOS | Homebrew | Commands/libraries in Homebrew dirs |

- One manager can't satisfy another scope's dependencies (pip can't install a system command; apt doesn't know about your venv).
- **One app can span several scopes at once**: e.g. a Python lib (pip), a system executable (PATH), a font (OS filesystem), a remote service (network) — installing the Python part proves nothing about the rest.
- **Repositories** = source of package metadata/files for an OS; differ by OS version/config, so same package name ≠ same result on different machines. Repos also change over time.
- OS packages often live in **shared directories** — upgrading for one app can break another. Modifying shared state often needs admin rights; project-owned envs usually don't.
- **Documentation ≠ isolation** — a setup guide still depends on each user's OS/repos/permissions/shared state. If this keeps causing failures → containers.

## 6.5 Processes, Filesystems & Isolation
- Compare tools via 3 pieces: the **process**, the **filesystem view**, the **kernel**.
- **Process** = running instance of a program (own memory, open files, env vars, permissions, cwd). Same program run twice = two separate processes.
- **Kernel** = OS core (hardware, memory, filesystems, networking, scheduling). Apps run in **user space**, calling the kernel when needed.
- **Host** = machine/environment providing the kernel + running the isolation tool. **Guest** = separate OS running on virtualised hardware.

### Three boundaries compared
| Tool | Separates | Still shared |
|---|---|---|
| Python venv | Python launcher + package location | processes, filesystem, system commands, kernel |
| Container | App processes + user-space filesystem view + config | Linux kernel (from host) |
| Virtual Machine | Complete guest OS on virtual hardware | physical host resources (via hypervisor) |

- **Hypervisor** = software that creates/manages virtual hardware for a VM.
- Choose based on **where the dependency lives**: only a Python package → venv; also needs system files/commands → container; needs a whole separate OS → VM.

### Image vs container
- **Image** = reusable package of user-space files + config (like a program file).
- **Container** = a running instance created from an image (like a process).
- Changes inside a running container do NOT change the original image. New containers always start fresh from the image.
- Data that must survive beyond a container needs an explicit external storage location.
- Containers can mount host files — but those stay host-controlled (can reintroduce path/permission differences).
- Isolation reduces variation, doesn't eliminate everything (network, secrets, live input still vary).

## 6.6 Descriptions, Recipes & Artifacts
- Three separate stages: **recorded input** (recipe) → **constructed state** (artifact) → **runtime instance** (running process).
- In Docker: **Dockerfile** (recipe) → **Image** (artifact) → **Container** (runtime instance).

| Layer | Recorded input | Constructed state | Runtime instance |
|---|---|---|---|
| Python | requirements/lockfile | venv + installed packages | Python process |
| Container | Dockerfile + build inputs | Image | Container |
| VM | VM config/provisioning | Virtual disk/machine image | Running guest OS |

- Recorded input = reviewable text. Constructed state = can be large/platform-specific. Not all constructed state is meant to be shared (a local venv is rebuilt each time; a tested image may be distributed).
- Sharing just the **recipe** → receiver must rebuild. Sharing the **image** → receiver runs the exact same built result. Registries store/distribute built images.

### Build time vs run time
- **Build time**: select base env, install deps, create dirs, copy source — becomes part of the image.
- **Run time**: choose what process starts in the container — uses files already captured, doesn't repeat install steps.

### Build context
- **Build context** = the directory of files available to the build; files outside it are unavailable.
- `.dockerignore` excludes unneeded/sensitive files (venvs, caches, temp files, credentials) → faster, safer builds.

### Why recipes can produce different results later
- Recipes reference changeable inputs (base image tag, OS repo, package range, remote download) → same Dockerfile can build a different image later.
- Reusing an already-built image = more repeatable than rebuilding. Still rebuild deliberately for security/dependency updates, then test before replacing.
- **Editing source later does NOT update an old image** — a build only copies source that existed at build time. Need a new build (or a runtime mount) to reflect changes.

### Build caching
- Builders reuse earlier steps if instructions/inputs unchanged (e.g. install deps before copying frequently-changing source, so dep install is cached).
- Caching speeds builds but does NOT auto-update an image — changed inputs still need a new build.

### Verify what actually ran
- Description = intent. Artifact = constructed contents. **Runtime evidence** = what a process actually got (version output, paths, exit status, test results).
- Practical flow: record inputs → build clean → run → inspect actual result.

## 6.8 Virtual Machines
- A **VM** runs a full guest OS + guest kernel on virtualised hardware — stronger/more complete boundary than a venv or container.

| Software | Common use |
|---|---|
| VirtualBox | Cross-platform teaching/testing |
| VMware Workstation/Fusion | Windows/Linux & macOS dev machines |
| Parallels Desktop | Windows/Linux guests on macOS |
| Hyper-V | Built into supported Windows editions/Server |
| QEMU (+ UTM on macOS) | Virtualization & architecture emulation |

- Use a VM when you need: a guest kernel/different OS, a stronger boundary for untrusted/risky software, to preserve a legacy OS, or a full server-like guest you can snapshot/restore.
- Costs: more memory, disk, startup time, OS maintenance than a container. Stronger boundary ≠ zero vulnerabilities.
- **Docker Desktop** on macOS/Windows runs a managed Linux VM under the hood (since Linux containers need a Linux kernel) — you interact with Docker, not the VM directly.

## 6.9 Recognising Another Package Ecosystem
- To approach any unfamiliar ecosystem, find 4 things: dependency declaration, resolved-version record, installed packages/cache, and the command that rebuilds project state.

| Ecosystem | Declaration | Resolved-version record | Install/build command |
|---|---|---|---|
| Python | pyproject.toml / requirements.txt | tool-dependent (pinned file may work) | `python -m pip install -r requirements.txt` |
| Node.js | package.json | package-lock.json | `npm ci` |
| Java (Maven) | pom.xml | no standard separate lockfile | `mvn package` |
| Rust | Cargo.toml | Cargo.lock | `cargo build` |

- Don't force every ecosystem into Python vocabulary — read that tool's own docs.
- Investigation steps: identify runtime/version → find dependency declaration → find resolved-version file → find clean install/build command → check what should stay out of Git.
- The transferable skill = recognizing **what state a file records**, not memorizing filenames.

## 6.10 Requirements Files vs Lockfiles
- Not interchangeable — different roles.
- Example pinned Python file: `requests==2.31.0` — human-maintained, exact direct dependency.
- **`pip freeze`**: dumps ALL currently installed packages (direct + transitive) in requirements format — doesn't distinguish which you intended vs which came along transitively.
- **Lockfile** (e.g. npm's): generated by the tool, records the full resolved dependency graph + tool-specific data for repeatable installs.

| Question | Pinned requirements.txt | Generated lockfile |
|---|---|---|
| Who edits it? | developer, directly | tool generates it |
| What it records | names/versions/URLs/markers (pip-supported) | exact structure per that ecosystem's tool |
| Records OS/Python runtime? | No | Usually not the full runtime either |

- An exact requirement improves repeatability for **that package only** — doesn't guarantee identical results across OS/Python version/CPU/index.
- Layered picture: requirements file → package choices; venv → isolates install location; Dockerfile → adds base image, system packages, files, paths, and command.

## 6.11 Week 6 Cheatsheet

| Problem | Tool/Record | Controls |
|---|---|---|
| Package version changes | `requirements.txt` | Declared Python package version |
| Two projects need different versions | `venv` | Separate package location per project |
| Program needs a Python library | `pip` | Packages for selected Python env |
| Program needs a system command | `apt-get` in image | System packages in image filesystem |
| Another machine needs the full setup | Dockerfile + image | Python base, packages, system tools, files, paths, command |

### Python environment commands
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt

python -c "import sys; print(sys.executable)"     # which interpreter
python -c "import requests; print(requests.__version__)"
python -c "import requests; print(requests.__file__)"  # which package path

deactivate
```

### Docker commands
```bash
docker build -t fit2109-week6 .
docker run --rm fit2109-week6
docker run --rm fit2109-week6 pwd
docker run --rm fit2109-week6 python inspect_env.py
```

### Core Dockerfile instructions
| Instruction | Purpose |
|---|---|
| `FROM` | Select base environment |
| `RUN` | Run a setup/install command during build |
| `WORKDIR` | Set stable internal working path |
| `COPY` | Copy files from build context into image |
| `CMD` | Set default runtime command |

### Key distinctions to remember
- **Dockerfile** = build recipe.
- **Image** = reusable built contents + config.
- **Container** = runtime instance from an image.
- **venv** = isolates Python packages only — no new OS/Python version.
- **`--rm`** = removes the container after exit, NOT the image.
- **Source change** = must rebuild the image; old image won't auto-update.