# Luxurious Python Stack — Documentation

The Luxurious Python Stack is a combination of tools, scripts, aliases, and rules for efficient, conflict-free Python development. Its five-level architecture separates concerns into distinct layers, each with specific tools and mechanisms. The use of ZSH with the plugin collection (Antidote) and the Powerlevel10K Prompt is not just for beautification, it also anchors functional extensions from the area of ​​data science (jupyter, mamba).

* For AI Agents
    > * The [skill](skills/luxuspythonstack/SKILL.md) provides structured instructions for AI coding agents working on projects initialized with the init of the stack.

* For Data Science
    > * The jupyter notebook [dsdash](scripts/dsdash.ipynb) contains some useful code snippets (Cuda test, Ollama Model Explorer).
    > * The data science environment includes cuda support for pytorch, tensorflow, numba, jax.

# Installation

* First clone the repo
```shell
mkdir -p ~/gits
git clone https://github.com/frankausberlin/luxus-skills ~/gits/luxus-skills
```
* and then, **follow the [setup instructions](Stack-Setup.md)**

> ⚠️ I recommend doing the setup in a VM or a fresh Ubuntu installation. I haven't tested any other distributions. If the system is to be integrated into an Ubuntu that is already in use, I recommend having this done by a good agent based on the setup instructions.

## Prerequisites

The Luxurious Python Stack requires the following installed at the system level:

| Tool | Purpose | Installation |
|------|---------|-------------|
| `uv` | Python version + package manager | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| `direnv` | Auto-activate virtual environments | `brew install direnv` or `sudo apt install direnv` |
| `just` | Command runner | `brew install just` or `cargo install just` |
| `ruff` | Linting + formatting | `uv tool install ruff` |
| `basedpyright` | Strict type checking | `uv tool install basedpyright` |
| `mamba` | Data science environments (Level 1) | Miniforge installer |

For a full Ubuntu Python setup, do the [python install guide](eco-systems/02-python.md). 

### direnv Hook

The direnv hook must be at the **absolute end** of your shell configuration:

```shell
# In ~/.zshrc or ~/.bashrc (must be the LAST line)
eval "$(direnv hook zsh)"   # or: eval "$(direnv hook bash)"
```

### direnv Utility Function

Create or verify `~/.config/direnv/direnvrc` with the `layout_uv` function:

```bash
layout_uv() {
    [ ! -d ".venv" ] && uv venv
    export VIRTUAL_ENV="$(pwd)/.venv" && PATH_add "$VIRTUAL_ENV/bin"
    export UV_PROJECT_ENVIRONMENT="$VIRTUAL_ENV"
}
```

## Shell Library Installation

This file is the canonical source for installation and update instructions. Other project files should link here instead of duplicating full command blocks. The [skill](skills/luxuspythonstack/SKILL.md) provides structured instructions for AI coding agents.

The recommended installation is a symbolic link. Do **not** copy the shell library into your shell setup. A symlink keeps the live repository as the source of truth, so updates are only:

```shell
cd /path/to/deen-lupysta
git pull
```

### Direct Shell Sourcing

For a minimal setup, source the repository file directly in your `.zshrc` or `.bashrc`:

```shell
source /path/to/deen-lupysta/scripts/luxuspythonstack.sh
```

### shlib Integration

If you use the [shlib system](base-system/02-zsh1-shlib.md), create a symlink in your shlib directory. Choose the two-digit prefix yourself according to your load order; `80` is only an example.

```shell
ln -s /path/to/deen-lupysta/scripts/luxuspythonstack.sh \
      /path/to/.shlib/shlibs/10-luxuspythonstack.sh
```

### Agent Skill Installation

Install the AI agent skills with symlinks as well. Link the repository folders into every agent-specific skills directory where the stack should be available:

```shell
ln -s /path/to/deen-lupysta/skills/luxuspythonstack \
      /path/to/agent/skills/luxuspythonstack
ln -s /path/to/deen-lupysta/skills/refactoring \
      /path/to/agent/skills/refactoring

# Repeat for additional agents if needed:
ln -s /path/to/deen-lupysta/skills/luxuspythonstack \
      /path/to/other-agent/skills/luxuspythonstack
ln -s /path/to/deen-lupysta/skills/refactoring \
      /path/to/other-agent/skills/refactoring
```

This keeps the shell library and all agent skill references updateable through the same repository checkout.

## Verify Installation

```shell
# All commands should be available
type pyinit act jl cw rlb
pyinit --help   # show usage without side effects
```

# Architecture — Five-Level Concept

The Luxurious Python Stack separates concerns into five distinct layers. Each level has specific tools and purposes, ensuring environments never conflict.

## Level Overview

| Level | Name | Tool | Activation |
|-------|------|------|------------|
| 0 | Global / System | `/usr/bin/python` | Always active (fallback) |
| 1 | Mamba / Jupyter | Mamba + UV | `act <envname>` or startup via `~/.startenv` |
| 2 | Projects | UV + direnv | Auto via `direnv` when `.venv` exists |
| 3 | CI / CD | GitHub Actions + UV | On push/PR or manual trigger |
| 4 | AI Agents | AGENTS.md + SESSION.md | Agent session start |

## Level 0: Global / System

- The standard Python installation lives here (`/usr/bin/python`)
- Core tools installed via system package manager: `uv`, `just`, `ruff` (via `uv tool`), `basedpyright`, `direnv`
- Mamba/conda is installed but inactive until explicitly activated
- The system level is the fallback when no other environment is active

## Level 1: Mamba / Jupyter

- Intended for data science and experimentation
- Tools: Jupyter Lab, PyTorch, TensorFlow, Scikit-Learn, etc.
- Always active unless a `.venv` exists at the current location (superseded by direnv)
- **Startup:** `act <envname>` activates the environment and saves it to `~/.startenv` (absolute path in home directory)
- **Volatile:** The rule is "no updates, just delete and recreate"
- **Naming convention:**<br> 
    - `ds12` = "data science Python 3.12"
    - `_ds12` = "ds12 with normal ipywidgets>=8.0"
- **The ipywidget dillema:**<br>
    Ipywidgets only work in colab in version <8.0. However, these are no longer supported by the current jupyter version.<br>
    The solution: I provide two versions of the data science environment for each Python version: a version with ipywidgets that can be used as a local runtime for Colab and a version that renders the widgets correctly in the local Jupyter (localhost:8888/lab).
- `uv pip install` is used for speed inside Mamba environments (intentionally not under Mamba's control)


## Level 2: Projects

- Standard Python projects managed with `uv` + `direnv`
- Activated automatically via `direnv` when a directory contains `.venv`
- Initialized with `pyinit` (application), `pyinit --lib` (library) or `pyinit --cli` (application with a Typer CLI reference)
- Quality gate: `just check` → `ruff check` + `ruff format --check` + `basedpyright` (+ `docs-check` when generated) + `pytest`; `just clean` removes local caches and generated artifacts; `just audit` → pip-audit
- Version bumping: `just bump patch|minor|major`

### Project Structure (created by pyinit)

```
src/<package_name>/
tests/
pyproject.toml
Justfile
AGENTS.md
.envrc
.pre-commit-config.yaml
.github/workflows/{ci,release}.yml
```

## Level 3: CI / CD

- Automated gatekeeper between local development and production
- Environment parity via `uv.lock`
- On every push/PR: automated `just check` runs
- Release automation: `bump-my-version` + Git tagging
- Automated PyPI publishing via GitHub Actions with OIDC Trusted Publishing

## Level 4: AI Agents

- `AGENTS.md` (tracked in git): project overview, conventions, key commands
- `SESSION.md` (gitignored, volatile): overwritten each session with current state
- `JOURNAL.md` (gitignored, append-only): accumulated session history
- Session workflow: read → work → check → commit → update session files
- Skill **luxuspythonstack** for coding agents.

## Level Interactions

Levels are layered and override each other:

```
Level 2 (.venv via direnv)
    overrides
Level 1 (Mamba)
    overrides
Level 0 (System Python)
```

When direnv detects a `.venv` in the current directory, it automatically deactivates the Mamba environment and activates the project's `.venv`. This ensures isolation without manual switching.


# Luxus Python Stack — Daily Commands

The complete command reference — project initialization, environment management,
dependencies, running code, testing, code quality, release, git workflow, shell
functions and troubleshooting — lives in
[skills/luxuspythonstack/references/daily-commands.md](skills/luxuspythonstack/references/daily-commands.md).

Optional end-to-end testing with real external tools in a disposable Docker
container is described in
[skills/luxuspythonstack/references/e2e-testing.md](skills/luxuspythonstack/references/e2e-testing.md).

## AI Agent Workflow (Level 4 — Vibe Coding)

`pyinit` generates each project's `AGENTS.md` from the marked export block below. The generated file is a concise project contract; project-local additions belong in its `Project-Specific Rules` section.

<!-- AGENTS_EXPORT_BEGIN: stack-contract-v1 -->
## Project Agent Contract

### Quality Commands

```bash
just check       # full gate: lint, formatting, type checking, and tests
just fix         # auto-fix lint and formatting
just test        # pytest with coverage
just lint        # ruff and basedpyright
just typecheck   # basedpyright only
just audit       # dependency vulnerability scan
just clean       # remove local caches and generated artifacts
just rate-design # write design metrics to .refactor/ (report only)
just refactor    # prepare a refactoring session (preflight, baseline, branch)
just merge       # fast-forward local main to a branch, remove its worktree and delete the branch (no push)
```

### Documentation (when generated)

- `just docs` builds the documentation; `just docs-check` verifies it is
  current and is part of `just check` when the recipe exists.
- Libraries document with MkDocs (`just docs-serve` for a live preview); apps
  created with `pyinit --cli` generate a CLI reference at
  `docs/reference/cli.md`.
- Generated pages are never edited by hand.

### Project Layout

- `src/<package>/` contains the package source.
- `tests/` contains the pytest suite and coverage checks.
- Applications expose their command through `src/<package>/main.py`.
- Libraries expose their public API through the package `__init__.py` and keep `py.typed`.

### Code Style

- Use the project's configured Python version and strict basedpyright checking.
- Add Google-style docstrings to public functions and classes.
- Keep lines at or below 120 characters.
- Let ruff sort imports and enforce the configured lint rules.
- Comments explain non-obvious decisions, not obvious code.

### Quality Rules

- Keep `just check` green; do not weaken tests, lint rules, or type checking to hide failures.
- Keep code free of dead code and unused imports.
- Chain exceptions with `raise ... from exc` when preserving the cause.
- Put tests in `tests/` and keep the configured coverage minimum (90%).

### Refactoring

- Measure before changing: `just rate-design` writes advisory radon/vulture
  reports to `.refactor/` (gitignored) and never fails on findings.
- Prepare a session with `just refactor [name]`: it refuses a dirty tree, runs
  `just check`, writes a baseline report and creates a `refactor/<slug>` branch.
  It changes no code.
- Then follow the `refactoring` skill: one recipe per step, `just check` green
  before and after every change, no test weakening, no destructive git.

### Session Workflow

1. Read `AGENTS.md`, then read `SESSION.md` if it exists.
2. Run `uv sync` and `git status` before making changes.
3. Work in small, tested changes and run `just check` before finishing.
4. Overwrite `SESSION.md` with the current state summary.
5. Append a dated entry to `JOURNAL.md`; never overwrite its history.

### Git Workflow

- Work on a branch (`feat/...`, `fix/...`, `docs/...`), preferably in a separate
  `git worktree`; keep `just check` green before merging.
- Merge from the `main` checkout with `just merge [branch]`: it fast-forwards
  `main` (refusing on divergence), removes the branch's worktree and deletes the
  branch. Without an argument it auto-detects the branch checked out in another
  worktree, else the only local branch not merged into `main`; it refuses when
  none or several match, and it never pushes.
- On divergence: rebase onto `main`, re-run `just check`, then merge again.
- Agents never push. The maintainer reviews local `main`
  (`git log --oneline origin/main..main`, `git diff origin/main..main`) and
  publishes with `git push origin main`.

### Debugging Protocol

1. Isolate or write a failing test first: `uv run pytest -k <name> -s`.
2. Read the complete traceback and identify whether the failure originates in `src/` or a dependency.
3. Use logging for complex state and `breakpoint()` for local inspection.

### Troubleshooting

| Symptom | Solution |
|---------|----------|
| `ModuleNotFoundError` | Run `uv sync` or use `uv run`. |
| `just check` fails | Run `just fix`, then resolve remaining failures manually. |
| `bump-my-version` fails | Ensure the working tree is clean. |

<!-- AGENTS_EXPORT_END: stack-contract-v1 -->

## Pyinit Flow

|pyinit|step|details|
|---:|:---|:---|
|01|argument parsing|A: `while`/`case` parser with `shift`; options may appear before or after the positional<br>B: `--lib` → library instead of app (default: app)<br>C: `--cli` → app with a Typer CLI reference (app only; `--lib --cli` is an error)<br>D: `--python X.Y` or `--python=X.Y` (default 3.12)<br>E: `--help` → print usage, exit 0, no side effects<br>F: at most one positional `DIRECTORY` (default `.`)<br>G: reject unknown options, `--lib --cli`, too many args, missing/empty values — before any change|
|02|version validation|A: Python version must match `X.Y` (e.g. 3.12), else abort|
|03|name derivation & validation|A: `PROJECT_NAME = basename <target>` (or `basename $PWD` for `.`)<br>B: `PACKAGE_NAME = PROJECT_NAME` with `-` → `_`<br>C: reject non-identifier names (leading digit, symbols, umlauts)<br>D: reject Python keywords (`class`, `import`, …) as package names|
|04|empty-target check|A: allow a non-existent target or a fully empty directory<br>B: reject a file-as-target<br>C: reject non-empty directories, including dotfiles and `.git/`<br>D: on rejection change nothing and keep the caller's cwd|
|05|tool & contract preflight|A: require `uv`, `just`, `git`, `awk` on PATH<br>B: resolve `luxuspythonstack.md` and extract exactly one non-empty `stack-contract-v1` export block before any target write; malformed or missing content aborts|
|06|enter target + uv init|A: run in a subshell so cwd and `UV_PYTHON_PREFERENCE` cannot leak<br>B: `UV_PYTHON_PREFERENCE=only-managed` scoped to the init<br>C: app → `uv init --app --package --name … --vcs none`; lib → `uv init --lib --name … --vcs none`|
|07|dev dependencies|A: `uv add --dev ruff pytest pytest-cov basedpyright colorlog bump-my-version pre-commit pip-audit radon vulture`<br>B: lib additionally `uv add --dev mkdocs-material "mkdocstrings[python]"`<br>C: app `--cli` additionally `uv add typer` (runtime dependency)|
|08|patch pyproject|A: app → `[project.scripts]` value `pkg:main` → `pkg.main:main`<br>B: add `# project-version` sentinel to the version line (bump-my-version needs it)|
|09|package source|A: `src/<pkg>/__init__.py` — docstring + `__version__ = "0.1.0"`<br>B: app → `src/<pkg>/main.py` with `main()` entry point<br>C: lib → `py.typed` kept (created by uv init)|
|10|tests|A: `tests/test_smoke.py` — app: import + run `main()`; lib: import + assert `__version__`<br>B: no `tests/__init__.py`, no `conftest.py`, no `assert True` placeholder|
|11|pyproject tool config|A: `[tool.ruff]` + lint select `E,F,W,I,N,D,UP,B,SIM,RUF`, google docstrings, line-length 120, `extend-exclude = [".kilo"]`<br>B: `[tool.basedpyright]` strict, `venvPath="."`, `venv=".venv"`<br>C: `[tool.pytest.ini_options]` `testpaths=["tests"]`, `addopts=["-ra","-q"]` — no `pythonpath` (package is installed)<br>D: `[tool.coverage.report]` `fail_under = 90`<br>E: `[tool.bumpversion]` + file mappings (`pyproject.toml`, `src/<pkg>/__init__.py`)|
|12|.gitignore|A: local deterministic template, no gitignore.io request<br>B: Python, env/secrets, tool caches, docs output, session files, `ignore/`, `.refactor/`, `.kilo/`<br>C: no `.vscode/` entry → settings/launch/extensions stay versionable|
|13|VS Code config|A: `settings.json` — basedpyright sole LSP, ruff formatter scoped to `[python]`, pytest enabled<br>B: `launch.json` — `debugpy` debugger type<br>C: `extensions.json` — `ms-python.python`, ruff, basedpyright|
|14|direnv|A: `.envrc` — conda guard, `watch_file uv.lock`, `layout uv` (documents `layout_uv` prerequisite)<br>B: no automatic `direnv allow`; printed as a manual next step|
|15|Justfile|A: common recipes — `test`, `lint`, `typecheck`, `check`, `fix`, `clean`, `audit`, `bump`, `rate-design`, `refactor`, `merge`; `check` runs `docs-check` when it exists (lib / app `--cli`)<br>B: app → `run` = `uv run <script>` with the command name fixed at generation time|
|16|GitHub Actions|A: `ci.yml` — `uv sync --locked --dev`, `just check`, `uv run pip-audit`<br>B: `release.yml` — libraries only; `check` job + `publish` job with `needs: check`, `uv build` + `uv publish`|
|17|AGENTS.md|A: generated before project docs from the `stack-contract-v1` export block<br>B: includes a generated project profile, app/library-specific entry details, managed contract markers, and an empty project-specific rules section<br>C: unresolved placeholders are rejected|
|18|session & project docs|A: `SESSION.md`, `JOURNAL.md` created once (later update is agent workflow)<br>B: `README.md`, `.editorconfig`, `CONTRIBUTING.md`, `LICENSE` (MIT)|
|19|pre-commit|A: `.pre-commit-config.yaml` — pre-commit-hooks + local ruff check/format<br>B: basedpyright excluded (too slow per commit; runs in `just check`/CI)|
|20|Docs (lib: MkDocs; app `--cli`: generated CLI reference)|A: lib → `mkdocs.yml` + `docs/index.md` + `docs/api.md`; `just docs` (`mkdocs build`), `just docs-check` (`mkdocs build --strict`), `just docs-serve` (live preview)<br>B: app `--cli` → `scripts/gen_cli_docs.py` + `docs/reference/cli.md`; `just docs` regenerates the page, `just docs-check` verifies it<br>C: generated pages are never edited by hand|
|21|git init|A: `git init -q -b main` (uv ran with `--vcs none`)|
|22|sync, hooks & quality gate|A: `uv sync`, `uv run pre-commit install`<br>B: lib / app `--cli` → `just docs` once (writes the generated page) before the gate<br>C: `just check` (includes `docs-check` for lib / app `--cli`)<br>D: report success only after all checks pass; print type-specific next steps|
