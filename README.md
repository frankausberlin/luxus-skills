# Luxus Skills

Reusable agent skills for a comfortable, consistent development workflow.

The first skill is **lupysta**, the Luxurious Python Stack: Python project scaffolding with `pyinit`, development tools, and guidance for both humans and coding agents. It is being extracted from [deen-lupysta](https://github.com/frankausberlin/deen-lupysta), with testing and design-review patterns adapted from [Lion](https://github.com/frankausberlin/lion).

## Current status

**Under development.** This repository currently contains the initial skill, reference documents, and shell script copied from the previous stack. Some paths still refer to the old repository, and the script expects a document that has not yet been migrated. Standalone project initialization is not ready yet.

The [roadmap](PROJECT_ROADMAP_de.md) describes the implementation stages and acceptance criteria. The [project notes](PROJECTNOTE_de.md) preserve the original ideas; the roadmap incorporates the subsequent decisions.

## How lupysta fits into a project

Install lupysta once in your agent harness. It provides the complete development and delivery workflow through a short overview and detailed recipes loaded when needed.

Each project's `AGENTS.md` records its local rules and the features available in that project. The agent uses this selection to find the relevant recipes in the installed skill, then checks the actual project commands before using them.

| Location | Responsibility |
| --- | --- |
| `lupysta/SKILL.md` | Workflow overview, shared rules, and recipe index |
| Skill references | Detailed setup and feature recipes, loaded on demand |
| Project `AGENTS.md` | Project context, local rules, and available features with recipe names |
| Project `Justfile` and scripts | Executable implementation of the selected features |

There is no second lupysta skill inside each generated project. The installed skill guides efficient agent work; generated commands use project-local files.

Projects can remove features they do not need. Their feature list and related configuration must then be updated, while the remaining workflow continues to work. Agents should load instructions relevant to the task and the available features, and should not recreate removed features automatically.

## Planned project features

The target scaffold covers:

- **Python projects:** applications, Typer CLI applications, and typed libraries managed with uv.
- **Everyday checks:** pytest with coverage, Ruff, basedpyright, and a local quality gate.
- **E2E tests:** an optional disposable-container suite under `tests/e2e/`, started with `just test-e2e`.
- **Agent tests:** an optional suite under `tests/agent/`, started with `just test-agent`, with bounded execution and diagnostic artifacts.
- **Design and refactoring:** design reports, saved baselines, comparisons, and refactoring preparation.
- **Documentation and delivery:** documentation commands appropriate to the project type, versioning, and CI configuration.
- **Environment setup:** guidance for the Python toolchain, optional Zsh comfort features, and optional Mamba/Jupyter environments.

Both additional test suites will start successfully with zero tests, without requiring Docker or a model provider. They remain outside ordinary tests, the default quality gate, and default CI. Adding real tests activates their documented prerequisites.

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

The central skill is maintained in this checkout. Updating it does not overwrite existing projects. `pyinit` initializes new projects; project-specific adaptations stay with those projects.

## Repository layout

| Path | Contents |
| --- | --- |
| [`lupysta/SKILL.md`](lupysta/SKILL.md) | Current skill entry point |
| [`lupysta/references/`](lupysta/references/) | Detailed guidance being reorganized into recipes |
| [`lupysta/scripts/`](lupysta/scripts/) | Bundled shell functions, including pyinit |
| [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md) | Implementation plan and acceptance criteria |
| [`PROJECTNOTE_de.md`](PROJECTNOTE_de.md) | Original project notes |

Scaffold templates are planned under `lupysta/resources/pyinit/`. The optional local `ignore/` directory is excluded from Git and is not required to install or use the skill.

## Development

Keep the skill overview concise and put detailed procedures in references. Keep generated features modular, document their prerequisites, and verify that removing an optional feature leaves the remaining workflow usable.

Code, skills, and regular documentation are written in **English**. Project notes, the roadmap, and agent conversations may be in German.
