---
name: luxuspythonstack
description: >
  Expert knowledge of the Luxurious Python Stack — a five-level development
  workflow for Python projects using UV, direnv, ruff, basedpyright, and AI
  agent integration. This skill should be used PROACTIVELY when working on
  Python projects initialized with pyinit, when the user needs project
  scaffolding, environment management, quality gate automation, version bumping,
  or AI agent session workflow guidance.
version: 0.4.0
license: MIT
platforms: [linux, darwin]
metadata:
  tags: [python, uv, direnv, ruff, basedpyright, quality-gate, pyinit, ai-agents]
  related_skills: [refactoring]
---

# Luxurious Python Stack

## Overview

This skill provides expert knowledge of the Luxurious Python Stack — a five-level development workflow designed for flexible, conflict-free Python environment management. The stack separates concerns into distinct levels, each with specific tools and purposes.


## Installation Model

The stack is installed by symbolic links, not by copying files. This keeps the repository checkout as the single source of truth and allows updates via `git pull`. The canonical installation guide is in the project root: [`luxuspythonstack.md`](../../luxuspythonstack.md).

Agents should recommend the canonical guide instead of duplicating full install commands. The key rules are:

- Link [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh) into the user's shlib setup with a user-chosen two-digit load-order prefix.
- Link the `skills/luxuspythonstack` folder into every agent-specific skills directory where this stack should be available.
- Avoid copy-based installation instructions.

## Maintenance / Update Policy

- Treat this repository checkout as the only editable source of truth.
- Do not patch files inside linked agent skills directories; edit the repository files instead.
- Do not replace symlinks with copied files during troubleshooting.
- Update installed integrations with `git pull` in the repository checkout, then reload the shell if shell functions changed.
- Keep detailed installation steps in [`luxuspythonstack.md`](../../luxuspythonstack.md); keep this skill file focused on agent behavior and operational rules.

## Five-Level Architecture

| Level | Name | Tool | Activation |
|-------|------|------|------------|
| 0 | Global / System | `/usr/bin/python` | Always active (fallback) |
| 1 | Mamba / Jupyter | Mamba + UV | `act <envname>` or startup via `~/.startenv` |
| 2 | Projects | UV + direnv | Auto via `direnv` when `.venv` exists |
| 3 | CI / CD | GitHub Actions + UV | On push/PR or manual trigger |
| 4 | AI Agents | AGENTS.md + SESSION.md | Agent session start |

**Key principle**: Levels are isolated. Level 2 (.venv) overrides Level 1 (Mamba) in any directory with a `.venv` folder via `direnv`.

## Decision Tree

```
Starting fresh in a project directory?
  → Has .venv? → Already at Level 2 (direnv auto-activates)
  → No .venv?  → Run pyinit (see ../../scripts/luxuspythonstack.sh) or uv init

Need to install a package?
  → In a project (.venv active): uv add <package>
  → In data science env:         uv pip install <package>
  → System-wide:                 avoid (use virtual envs)

Need to run code?
  → In terminal with direnv:  python src/...  (env auto-active)
  → In scripts/CI:            uv run python src/...  (guaranteed sync)

Need to release?
  → just bump [patch|minor|major]
  → git push origin main --tags
```

## Core Workflows

### 1. Initialize a New Python Project

Use the `pyinit` shell function (defined in [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh)):

```shell
pyinit                              # app in current directory
pyinit my-project                   # app in new directory
pyinit my-lib --lib                 # library in new directory
pyinit my-app --cli                 # app with a generated Typer CLI reference
pyinit my-project --python 3.11     # specify Python version (default: 3.12)
pyinit --help                       # show usage (no side effects)
```

`pyinit` only initializes empty or new directories; it is not an updater.
`--cli` is app-only: `pyinit --lib --cli` is rejected.

The script creates:
- `src/<package>/` with `__init__.py`, `main.py` (app), or `py.typed` (lib)
- `tests/test_smoke.py` (a real smoke test, not `assert True`)
- `pyproject.toml` with ruff, basedpyright, pytest, a 90% coverage gate, and bump-my-version config
- `.venv/` virtual environment (Python 3.12 by default)
- `.python-version`, `.vscode/` (settings, launch, extensions)
- `.envrc` for direnv auto-activation (with Conda deactivation guard)
- `.gitignore` (local template, no network request)
- `.editorconfig`, `Justfile`, `.pre-commit-config.yaml`
- `.github/workflows/ci.yml` (always) and `release.yml` (libraries only)
- `AGENTS.md`, `SESSION.md`, `JOURNAL.md`, `README.md`, `CONTRIBUTING.md`, `LICENSE`
- **Library projects only:** `mkdocs.yml` + `docs/` with Material theme and mkdocstrings; `just docs-check` (`mkdocs build --strict`) is part of `just check`
- **App `--cli` only:** `src/<package>/cli.py` (Typer app), `scripts/gen_cli_docs.py` and a generated `docs/reference/cli.md`; `just docs-check` is part of `just check`

After running pyinit, run `direnv allow` once (a manual step) to activate the `.venv` automatically.

### 2. Environment Management

**Mamba environment (Level 1)**:
```shell
act ds12          # activates and saves to ~/.startenv
mamba activate ds12  # without saving
```

**Project environment (Level 2)**:
```shell
# direnv handles activation automatically when entering the directory
# If module errors arise after git pull:
uv sync
```

**Recreate Mamba environment**:
```shell
py=3.12 && ENV_NAME="ds${py: -2}"
mamba deactivate
mamba remove -y -n $ENV_NAME --all 2>/dev/null
mamba create -y -n $ENV_NAME python=$py <packages...>
mamba activate $ENV_NAME
```

### 3. Daily Development Commands

```shell
uv add <package>          # runtime dependency
uv add --dev <package>    # dev-only dependency
uv sync                   # sync environment

just run                  # run the main script
just test                 # pytest with coverage
just lint                 # ruff + basedpyright
just typecheck            # type checking only
just check                # full quality gate (lint + typecheck + docs + tests)
just fix                  # auto-fix
just clean                # remove caches and generated artifacts
just audit                # security scan
just rate-design          # design metrics → .refactor/ (report only)
just refactor             # prepare a refactoring session (preflight, baseline, branch)
just merge                # fast-forward main to a branch and clean up its worktree/branch (no push)
```

When documentation is generated (`--lib` or app `--cli`):

```shell
just docs                 # build MkDocs (lib) / regenerate docs/reference/cli.md (app --cli)
just docs-check           # verify generated documentation is current (part of `just check`)
just docs-serve           # live preview (libraries only)
```

### 4. Release & Deployment

Never edit versions or create tags manually. Always use `bump-my-version`:

```shell
just bump patch           # 0.2.0 → 0.2.1
just bump minor           # 0.2.1 → 0.3.0
just bump major           # 0.3.0 → 1.0.0
git push origin main --tags
```

### 5. AI Agent Session Workflow (Level 4)

AI agents operate through MCP servers (web search, GitHub, Docker, browser automation, etc.) managed by MCPHub. Server setup is documented in the [AI stack](../../ai-stack/) section — load only when configuring agent infrastructure.

**Start:**
1. Read `AGENTS.md` for project context
2. Read `SESSION.md` if it exists
3. Run `uv sync` and `git status`
4. Begin work

**End:**
1. Run `just check` — must be green
2. **Overwrite** `SESSION.md` with current state (volatile)
3. **Append** dated entry to `JOURNAL.md` (history, never overwrite)
4. Commit all changes on the branch
5. From the main checkout, merge locally with `just merge [branch]` (fast-forward
   only; it removes the branch's worktree and deletes the branch) — never push

## Key Shell Functions

These shell helpers live in different files of the repository:

| Function | Usage | Defined in | Description |
|----------|-------|------------|-------------|
| `pyinit` | `pyinit [name] [--lib] [--cli] [--python X.Y] [--help]` | [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh) | Create a new Python project |
| `jupyter-launcher` | `jupyter-launcher [-n] [-c] [-p PORT] [folder]` | [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh) | Start Jupyter Lab |
| `jl` | `jl` | [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh) | Alias for `jupyter-launcher` (default: `-n -c`) |
| `act` | `act <envname>` | [`eco-systems/02-python.md`](../../eco-systems/02-python.md) | Activate Mamba env + save to `~/.startenv` |
| `cw` | `cw` / `cw .` | [`deenlupysta.sh`](../../scripts/deenlupysta.sh) | Jump to / set working folder (global state) |
| `rlb` | `rlb` | [`deenlupysta.sh`](../../scripts/deenlupysta.sh) | Reload the shell (`exec zsh`) |

`jupyter-launcher` is the canonical implementation. `jl` is intentionally only an alias so users can preconfigure personal defaults, for example `alias jl='jupyter-launcher -n -c'`.

## Important Rules

1. **`uv run` vs direct execution**: In scripts and CI always use `uv run` (auto-syncs). In terminal with direnv active, direct `python` is fine.

2. **Never mix uv packages into Mamba**: In Level 1, `uv pip install` is intentionally not under Mamba's control. The data science environment is disposable and rebuilt regularly.

3. **Version bumping**: Never edit `version =` in `pyproject.toml` or `__init__.py` manually. Never `git tag` manually. Always use `just bump`.

4. **direnv**: If auto-activation stops working, run `direnv allow` in the project directory.

5. **Generated documentation**: Never edit `docs/reference/cli.md` or other generated pages by hand. Run `just docs` and commit the result; `just check` fails when a generated page is stale.

6. **Git workflow**: Work on a branch (`feat/...`, `fix/...`, `docs/...`), keep `just check` green, then merge locally with `just merge [branch]` from the main checkout. Agents never push; the maintainer reviews local `main` and publishes with `git push origin main`.

## Reference Files

Load on demand as needed:

- [`references/daily-commands.md`](references/daily-commands.md) — Complete command reference
- [`references/e2e-testing.md`](references/e2e-testing.md) — Optional Docker-isolated end-to-end testing pattern
- [`../refactoring/SKILL.md`](../refactoring/SKILL.md) — Safe, measurable refactoring workflow (`rate-design`/`refactor`) and recipe catalog
- [`../../base-system/`](../../base-system/) — Full developer environment setup guides
- [`../../ai-stack/`](../../ai-stack/) — AI stack component setup
- [`../../luxuspythonstack.md`](../../luxuspythonstack.md) — Full documentation and canonical `stack-contract-v1` export for generated `AGENTS.md` files
