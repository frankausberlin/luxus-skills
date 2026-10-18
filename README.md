# Luxus Skills

Reusable agent skills for a comfortable, consistent development workflow.

The first skill is **lupysta**, the Luxurious Python Stack: Python project scaffolding with `pyinit`, development tools, and guidance for both humans and coding agents. It is being extracted from [deen-lupysta](https://github.com/frankausberlin/deen-lupysta), with testing and design-review patterns adapted from [Lion](https://github.com/frankausberlin/lion).

## Current status

**Under development.** The repository contains the initial skill, reference documents, and the shell script copied from the previous stack. The first review found that standalone project initialization is not ready yet: `pyinit` still expects a contract document that has not been migrated, several paths point at the old layout, and the platform claim is not backed by the implementation.

The [review](PROJECT_REVIEW_de.md) documents the findings and the recommended lean MVP. Stage 1 of the [roadmap](PROJECT_ROADMAP_de.md) — making the skill standalone and reliable — is the next step. The [project notes](PROJECTNOTE_de.md) preserve the original ideas.

## Project documents

| Document | Contents | Language |
| --- | --- | --- |
| [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md) | Stages, acceptance criteria, risks, open decisions | German |
| [`PROJECT_REVIEW_de.md`](PROJECT_REVIEW_de.md) | Findings R1–R15 with evidence and recommendations | German |
| [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md) | Decision log D1–D15 with status, options, consequences | German |
| [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md) | Feature model, file ownership, removal, `lupysta-status` | German |
| [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md) | Test and CI strategy, acceptance matrix | German |
| [`PROJECTNOTE_de.md`](PROJECTNOTE_de.md) | Original project notes | German |

Project-management documents are written in German. Code, the skill, and regular documentation such as this README and the skill references are written in English.

## How lupysta fits into a project

Install lupysta once in your agent harness. It provides the complete development and delivery workflow through a short overview and detailed recipes loaded when needed.

Each project's `AGENTS.md` records its local rules and the features available in that project, using a feature table with status, entry point, recipe name, and verification command. A planned `just lupysta-status` command checks that the declared entry points exist; it reports drift and never repairs anything. The agent uses this information to find the relevant recipes in the installed skill, then checks the actual project commands before using them. The generated `AGENTS.md` also records `lupysta_version` and `contract_version`.

| Location | Responsibility |
| --- | --- |
| `lupysta/SKILL.md` | Workflow overview, shared rules, and recipe index |
| Skill references | Detailed setup and feature recipes, loaded on demand |
| Project `AGENTS.md` | Project context, local rules, feature table with status and entry points |
| Project `Justfile` and scripts | Executable implementation of the selected features |

There is no second lupysta skill inside each generated project. The installed skill guides efficient agent work; generated commands use project-local files.

Projects can remove features they do not need. Their feature table, configuration, dependencies, and documentation must then be updated, while the remaining workflow continues to work. Removed features stay visible with status `removed`, so agents do not recreate them automatically. The ownership rules and removal checklist are in [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md).

## Planned project features

The target scaffold covers:

- **Core:** Python applications, Typer CLI applications, and typed libraries managed with uv, with `just check` as the entry point.
- **Design and refactoring:** a lean design report with saved baselines and comparisons, plus refactoring preparation in a separate worktree.
- **E2E tests:** an optional disposable-container suite under `tests/e2e/`, started with `just test-e2e`.
- **Agent tests:** an optional suite under `tests/agent/`, started with `just test-agent`, with bounded execution, provider preflight, and diagnostic artifacts.
- **Documentation:** `docs-lib` for libraries and `docs-cli` for CLI applications, with `just docs` and `just docs-check`.
- **Environment setup:** documented, optional `setup-zsh` and `setup-mamba` guidance for the Python toolchain, Zsh comfort features, and Mamba/Jupyter environments.

Both optional test suites start successfully with zero tests, without requiring Docker or a model provider. They are kept out of pytest collection by a root `tests/conftest.py` gate, not by markers alone, and remain outside ordinary tests, the default quality gate, and default CI. Adding real tests activates their documented prerequisites; missing prerequisites then fail loudly instead of being skipped.

These are the intended capabilities, not a claim that the migration is complete.

## Installation model

The following describes the target installation flow. Complete it once the standalone migration is ready.

1. Clone this repository to a stable location and link the `lupysta/` directory into the skills directory used by your harness. Use a name supported by that harness; `lupysta` is the intended skill name.
2. Obtain the generated `source` command containing the absolute path to `lupysta/scripts/luxuspythonstack.sh`. Run it in your shell or add it to your preferred shell configuration.

For example, with paths adjusted to your checkout and harness:

```sh
ln -s "/absolute/path/to/luxus-skills/lupysta" "/absolute/path/to/harness/skills/lupysta"
source "/absolute/path/to/luxus-skills/lupysta/scripts/luxuspythonstack.sh"
```

You choose where the shell command belongs. A particular shlib layout is not required. Loading the agent skill and sourcing the shell functions are separate steps.

For non-interactive shells, a small wrapper that sources `luxuspythonstack.sh` before calling `pyinit` is planned; it is optional and does not replace the `source` step for interactive use.

The script file keeps the name `luxuspythonstack.sh` for compatibility; the skill name is `lupysta`.

### Migrating from the old skill

- Do not install the old `luxuspythonstack` skill and lupysta at the same time. Remove the old skill link from every harness before linking `lupysta`.
- Existing projects created by the old `pyinit` are not updated automatically. Keep them as they are and use their existing project commands.
- To uninstall, remove the skill link from the harness and remove the `source` line from your shell configuration. Generated projects are independent of the installed skill.

### Supported environment

Linux is the reference platform. macOS is currently not a supported claim: several GNU-specific tools are still used in the script. macOS support returns only with CI and test coverage (see decision D2 in [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md)).

The central skill is maintained in this checkout. Updating it does not overwrite existing projects. `pyinit` initializes new projects; project-specific adaptations stay with those projects.

## Repository layout

| Path | Contents |
| --- | --- |
| [`lupysta/SKILL.md`](lupysta/SKILL.md) | Current skill entry point |
| [`lupysta/references/`](lupysta/references/) | Detailed guidance being reorganized into recipes |
| [`lupysta/scripts/`](lupysta/scripts/) | Bundled shell functions, including pyinit |
| `lupysta/resources/pyinit/` | Planned scaffold templates, including the `AGENTS.md` contract and feature table |
| `tests/` | Planned test suite for the skill repository |
| `.github/workflows/` | Planned CI: tests, shellcheck, link check, fresh-clone test |
| [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md) | Implementation plan and acceptance criteria |
| [`PROJECT_REVIEW_de.md`](PROJECT_REVIEW_de.md) | Review findings and recommendations |
| [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md) | Decision log |
| [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md) | Feature model and removal rules |
| [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md) | Test and CI strategy |
| [`PROJECTNOTE_de.md`](PROJECTNOTE_de.md) | Original project notes |

The optional local `ignore/` directory is excluded from Git and is not required to install, use, or test the skill. No document or script may depend on it.

## Development

Keep the skill overview concise and put detailed procedures in references. Keep generated features modular, document their prerequisites, and verify that removing an optional feature leaves the remaining workflow usable.

Before a change is merged:

- Run the fast test suite (`bash tests/test-pyinit.sh --fast` once it has been migrated).
- Run shellcheck on the shell files and the link checker on all Markdown files.
- Verify the fresh-clone test: a checkout without `ignore/` must install the skill and initialize a project.
- Check that every generated project's `AGENTS.md` feature table matches the files and recipes that were actually created.

The full strategy, including E2E and agent suites, is in [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md).

A `LICENSE` file and, if needed, a `NOTICE` for third-party parts are still missing. The license choice is open; the review recommends Apache-2.0 with attribution for MIT parts. See decision D3 in [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md).

Code, skills, and regular documentation are written in **English**. Project notes, the roadmap, the review, decisions, features, testing, and agent conversations may be in German.
