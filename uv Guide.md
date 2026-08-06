# The Complete Guide to `uv` (and How It Compares to Other Python Tools)

*Last updated: July 2026*

---

## 1. What is `uv`?

`uv` is an extremely fast Python package and project manager written in Rust by **Astral**, the same team behind the `ruff` linter. Instead of being "just another pip alternative," uv aims to replace the *entire* stack of tools Python developers have historically stitched together:

- `pip` (installing packages)
- `pip-tools` (lockfiles / compiling requirements)
- `virtualenv` / `venv` (environment creation)
- `pyenv` (Python version management)
- `poetry` / `pipenv` (project & dependency management)
- `twine` (publishing packages)

One binary, one cache, one lockfile format — that's the pitch. As of early 2026, Astral was acquired by OpenAI, giving uv unusually strong corporate backing for a developer-tooling project, and it has become the de facto default for new Python projects in much of the ecosystem.

### Why it's fast
- Written in Rust with a highly parallel resolver and downloader
- Global, content-addressed cache — the same wheel is never downloaded or stored twice
- Overlaps metadata fetching, dependency resolution, and disk writes instead of doing them sequentially
- Uses hardlinks/copy-on-write from the cache into virtual environments instead of re-extracting archives

Benchmarks vary by machine and network, but the order of magnitude is consistent: cold installs that take pip 20+ seconds or Poetry 15+ seconds often complete in uv in 1-3 seconds, and warm-cache operations are close to instantaneous.

---

## 2. Installation

**macOS / Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows (PowerShell):**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Via pip (if you already have Python):**
```bash
pip install uv
```

**Via pipx:**
```bash
pipx install uv
```

**Via Homebrew:**
```bash
brew install uv
```

Verify:
```bash
uv --version
```

Update uv itself at any time with:
```bash
uv self update
```

---

## 3. Core Concepts

uv operates in two broad modes:

1. **Project mode** — the "batteries included" workflow built around `pyproject.toml` and a `uv.lock` file. This is the recommended way to manage applications and libraries.
2. **Pip-compatible mode** (`uv pip ...`) — a drop-in, much faster replacement for `pip`/`pip-tools` commands, for teams not ready to adopt the full project workflow.

---

## 4. Managing Python Versions

uv can download and manage Python interpreters itself — no separate `pyenv` needed.

```bash
uv python install 3.12          # install a specific version
uv python install 3.10 3.11 3.12  # install several
uv python list                  # see installed/available versions
uv python pin 3.12              # pin a project to a version (.python-version file)
uv python find 3.11             # locate an interpreter
uv python uninstall 3.10
```

---

## 5. Starting and Managing a Project

```bash
uv init my-project        # scaffold a new project (pyproject.toml, .python-version, README)
cd my-project

uv add requests           # add a dependency (updates pyproject.toml + uv.lock)
uv add --dev pytest ruff  # add dev-only dependencies
uv remove requests        # remove a dependency

uv sync                   # install exact versions from uv.lock into .venv
uv lock                   # (re)generate the lockfile without installing
uv lock --upgrade         # upgrade all dependencies to latest allowed versions

uv run python script.py   # run a command inside the project's environment
uv run pytest             # no need to manually activate the venv
```

uv automatically creates and manages a `.venv` folder for the project — you rarely need to think about virtual environments directly, though `uv venv` is available if you want one manually.

### The lockfile: `uv.lock`
- Cross-platform and deterministic — the same lockfile resolves consistently on macOS, Linux, and Windows.
- Should be committed to version control.
- uv also has growing support for **PEP 751's `pylock.toml`**, the emerging cross-tool standard lockfile format, which improves portability between uv, pip, and Poetry over time.

---

## 6. Pip-Compatible Mode

For teams that want the speed without changing workflows:

```bash
uv venv                          # create a virtual environment
source .venv/bin/activate        # activate as usual

uv pip install requests          # drop-in replacement for `pip install`
uv pip install -r requirements.txt
uv pip compile requirements.in -o requirements.txt   # like pip-tools' `pip-compile`
uv pip sync requirements.txt     # like pip-tools' `pip-sync`
uv pip freeze
uv pip list
uv pip uninstall requests
```

Almost every standard `pip` flag works unchanged, so migrating is usually a find-and-replace of `pip` → `uv pip`.

---

## 7. Tools and Scripts

**Run a CLI tool without installing it globally (like `pipx run`):**
```bash
uv tool run ruff check .
uvx ruff check .          # `uvx` is a shorthand alias for `uv tool run`
```

**Install a tool persistently:**
```bash
uv tool install ruff
uv tool list
uv tool uninstall ruff
```

**Single-file scripts with inline dependencies (PEP 723):**
```bash
# script.py
# /// script
# dependencies = ["requests"]
# ///
import requests
print(requests.get("https://example.com").status_code)
```
```bash
uv run script.py    # uv creates an ephemeral environment just for this script
```

---

## 8. Building and Publishing Packages

```bash
uv build            # build sdist + wheel into dist/
uv publish          # upload to PyPI (or a configured index)
```

---

## 9. Workspaces (Monorepos)

uv supports **workspaces**, similar to Cargo workspaces in Rust — a way to manage multiple related packages with a single shared lockfile:

```toml
# root pyproject.toml
[tool.uv.workspace]
members = ["packages/*"]
```

---

## 10. Migrating From Other Tools

| From | How |
|---|---|
| **pip / requirements.txt** | `uv add -r requirements.txt`, or keep using `uv pip install -r requirements.txt` |
| **Poetry** | The community `migrate-to-uv` tool converts `[tool.poetry]` sections to the standard `[project]` table automatically |
| **pyenv** | No change needed — `uv python install` covers version management going forward |
| **pipenv** | Import your `Pipfile` dependencies into `pyproject.toml` and run `uv sync` |
| **conda** | uv doesn't manage non-Python dependencies (compilers, CUDA, system libraries); keep conda/mamba for that layer and use uv for pure-Python packages inside the env |

---

## 11. Comparison: uv vs. Other Tools

### Summary table

| Tool | Category | Speed | Lockfile | Python version mgmt | Env creation | Best for |
|---|---|---|---|---|---|---|
| **uv** | All-in-one | Fastest (10–100x pip) | Native `uv.lock`, PEP 751-aware | ✅ built-in | ✅ built-in | New projects, CI speed, monorepos, replacing the whole toolchain |
| **pip** | Installer only | Slowest of the mainstream tools | ❌ none natively | ❌ | ❌ (needs venv) | Universal baseline — ships with Python, everyone knows it |
| **Poetry** | Project manager | Moderate (~10x slower than uv) | Native `poetry.lock` | ❌ (plugin-based) | ✅ built-in | Library publishing, teams already invested in it |
| **Pipenv** | Project manager | Slow | `Pipfile.lock` | ❌ | ✅ built-in | Legacy projects; largely superseded by uv/Poetry |
| **pip-tools** | Lockfile compiler | Slow–moderate | `requirements.txt` (compiled) | ❌ | ❌ | Teams wanting minimal abstraction on top of pip |
| **conda / mamba** | Environment + package manager | Moderate (mamba is fast) | Environment YAML | ✅ (any language) | ✅ built-in | Data science / ML with non-Python (C, CUDA, Fortran) dependencies |
| **pyenv** | Python version manager only | N/A | N/A | ✅ (its only job) | ❌ | If you specifically don't want an all-in-one tool |
| **PDM** | Project manager | Fast | `pdm.lock`, PEP 582 support | ✅ (plugin) | ✅ | PEP 582-style local packages, standards-focused teams |
| **Hatch** | Project manager | Moderate | N/A (uses pip resolver) | ✅ (via hatch-python) | ✅ | Multi-environment testing matrices, plugin-driven builds |
| **Rye** | Project manager | Fast | `requirements.lock` | ✅ | ✅ | Now being merged into/converging with uv (same creator, Astral) |

### uv vs pip
pip is the installer that ships with Python — it's the one tool every environment can assume exists, and it's what most tutorials and Stack Overflow answers reference. But pip alone doesn't give you lockfiles, Python version management, or environment creation — you assemble those yourself from pip-tools, pyenv, and venv. uv absorbs all of that into a single fast tool, but pip remains the universal fallback and the safest choice for teaching or maximum compatibility. uv even includes `uv pip` specifically so teams can adopt its speed without leaving the pip mental model.

### uv vs Poetry
Both are full project managers built around `pyproject.toml` with native lockfiles. Poetry is mature, has a very polished PyPI publishing workflow, and is well understood by many teams. uv is substantially faster and additionally handles Python version installation, which Poetry doesn't do natively. If you're starting a new project, uv is generally the stronger default; if you have an existing healthy Poetry setup, there's no urgency to migrate unless you have a specific pain point (slow CI, multi-Python-version needs, monorepo workspaces).

### uv vs Pipenv
Pipenv pioneered the `Pipfile`/`Pipfile.lock` approach but has slower resolution and has seen less active development in recent years relative to uv and Poetry. Most new projects choose uv or Poetry instead.

### uv vs pip-tools
pip-tools (`pip-compile` / `pip-sync`) adds lockfile-style reproducibility on top of plain pip without adopting a new project format. uv replicates this exact workflow via `uv pip compile` / `uv pip sync`, but much faster, while also offering the option to move to the richer `uv add`/`uv sync` project workflow later.

### uv vs conda/mamba
This is the one comparison that isn't a straight substitution. Conda (and its faster reimplementation, mamba) manage more than Python packages — they can install compilers, CUDA toolkits, and other system-level dependencies, which matters heavily in data science and ML. uv only manages Python packages and interpreters. The common 2026 pattern: use conda/mamba to set up the environment and any non-Python dependencies, then use uv or pip to manage the pure-Python packages inside it.

### uv vs pyenv
pyenv does exactly one thing — install and switch between Python versions — and does it well via shell shims. uv's `uv python install/pin` covers the same need natively, so most people migrating to uv can simply stop using pyenv without any transition steps.

### uv vs PDM / Hatch / Rye
- **PDM** is standards-focused (an early adopter of PEP 582 local package directories) and reasonably fast, but has a smaller ecosystem than uv.
- **Hatch** is popular for its plugin system and multi-environment test matrices (similar to `tox`), but isn't primarily chasing raw speed.
- **Rye** was an earlier Astral project with similar all-in-one goals; since Astral also created uv, the two are converging, with uv increasingly the flagship going forward.

---

## 12. Decision Guide

| Situation | Recommendation |
|---|---|
| Starting a brand-new Python project | **uv** |
| Existing pip + requirements.txt project, want speed with minimal change | **uv pip** (drop-in mode) |
| Existing Poetry project that works fine | Stay on **Poetry** unless you hit a specific pain point |
| Data science / ML with GPU or system-level deps | **conda/mamba** for the environment, **uv** for Python packages |
| Publishing a library, team already knows Poetry | **Poetry** still has the smoothest publish flow, though uv is closing the gap |
| Teaching/tutorials aimed at total beginners | **pip** — matches official docs and most existing material |
| Monorepo with multiple interdependent Python packages | **uv workspaces** |
| One-off script needing a couple of packages | `uv run script.py` with inline PEP 723 dependencies |

---

## 13. Quick Reference Cheat Sheet

```bash
# Setup
uv python install 3.12
uv init myproj && cd myproj

# Dependencies
uv add <package>
uv add --dev <package>
uv remove <package>
uv sync
uv lock --upgrade

# Running things
uv run python main.py
uv run pytest
uvx ruff check .

# Pip-compatible mode
uv venv
uv pip install <package>
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt

# Tools
uv tool install <tool>
uv tool run <tool>

# Build & publish
uv build
uv publish

# Maintenance
uv self update
uv cache clean
```

---

## 14. Key Takeaways

- uv consolidates pip, pip-tools, virtualenv, pyenv, and much of what Poetry/Pipenv do — into one Rust-based binary.
- It is dramatically faster than every alternative, especially on cold installs and CI pipelines.
- It's not a strict replacement for conda in GPU/data-science contexts where non-Python system dependencies matter.
- Migration risk is low: `uv pip` mirrors pip syntax, and community tools exist to convert Poetry/Pipenv configs automatically.
- For new projects in 2026, uv is the sensible default; for existing healthy setups on Poetry or pip, there's no obligation to switch immediately.