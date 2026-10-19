# Luxus Python Stack — Daily Commands

Quick reference for common operations. For full documentation, see [`luxuspythonstack.md`](../../../luxuspythonstack.md).

## Project Initialization

```shell
pyinit                          # app in current directory
pyinit my-project               # app in new directory
pyinit my-lib --lib             # library in new directory
pyinit my-app --cli             # app with a generated Typer CLI reference
pyinit my-project --python 3.11 # specify Python version (default: 3.12)
pyinit --help                   # show usage (no side effects)
```

`pyinit` only initializes empty or new directories; it is not an updater.
`--cli` is app-only (`pyinit --lib --cli` is rejected). After initialization,
run `direnv allow` once to activate the `.venv` automatically.

## Environment Management

```shell
# Mamba (Level 1 — Data Science)
act <envname>             # activate + save to ~/.startenv
mamba activate <envname>  # activate without saving
mamba deactivate          # deactivate
jupyter-launcher [folder]         # Jupyter Lab (token enabled)
jupyter-launcher -n [folder]      # Jupyter Lab without token (unsafe)
jupyter-launcher -c [folder]      # Jupyter Lab with Colab origin
jupyter-launcher -n -c [folder]   # tokenless + Colab
jl                                # alias: jupyter-launcher -n -c

# UV/direnv (Level 2 — Projects)
direnv allow   # allow .envrc (once per project)
uv sync        # sync after git pull / dependency change
```

## Dependencies

```shell
uv add <package>          # runtime dependency
uv add --dev <package>    # dev-only dependency
uv remove <package>       # remove
uv sync                   # sync .venv with lock file
uv pip install <package>  # Level 1 only
```

## Running Code

```shell
just run                            # run the installed command (apps)
uv run python -m <pkg>.main         # guaranteed sync (CI/scripts)
uvx <tool> <args>                   # ephemeral tool run
```

## Testing

```shell
just test                       # all tests with coverage
pytest -v                       # verbose
pytest -k "test_name" -s        # specific test, show stdout
```

For opt-in end-to-end tests with real tools in a disposable Docker container,
see [`e2e-testing.md`](e2e-testing.md).

## Code Quality

```shell
just lint         # ruff + basedpyright
just typecheck    # type checking only
just check        # full gate (lint + typecheck + docs + tests)
just fix          # auto-fix
just clean        # remove caches and generated artifacts
just audit        # security: pip-audit
```

## Documentation

Generated only for libraries and apps created with `--cli`:

```shell
just docs         # build MkDocs (lib) / regenerate docs/reference/cli.md (app --cli)
just docs-check   # verify generated documentation is current (part of `just check`)
just docs-serve   # live preview (libraries only)
```

Never edit a generated page by hand; run `just docs` and commit the result.

## Refactoring

```shell
just rate-design  # design metrics → .refactor/ (report only)
just refactor     # prepare a refactoring session (preflight, baseline, branch)
```

`rate-design` never fails on findings; `refactor` refuses a dirty tree, runs
`just check`, writes the baseline and creates a `refactor/<slug>` branch without
changing code. Then follow the
[refactoring skill](../../refactoring/SKILL.md): measure first, one recipe per
step, `just check` green before and after.

## Version & Release

```shell
just bump patch   # 0.1.0 → 0.1.1
just bump minor   # 0.1.1 → 0.2.0
just bump major   # 0.2.0 → 1.0.0
git push origin main --tags
```

## Git Workflow

Work on a branch (`feat/...`, `fix/...`, `docs/...`), preferably in a separate
worktree. Keep `just check` green before merging:

```shell
git worktree add ../<project>-feature -b feat/my-feature
# ... make changes ...
just check
git add -A && git commit -m "feat: ..."

# After git pull: uv sync
```

From the `main` checkout, merge the branch locally (fast-forward only):

```shell
just merge feat/my-feature   # explicit branch
just merge                   # or auto-detect the branch in another worktree
```

`just merge` fast-forwards `main` to the branch and refuses when `main` has
diverged; it then removes the branch's worktree and deletes the branch. Without
an argument it auto-detects the branch checked out in another worktree, else the
only local branch not merged into `main`, and refuses when none or several match.
It never pushes.

On divergence, rebase onto `main`, re-run `just check`, and merge again:

```shell
git rebase main
just check
just merge feat/my-feature
```

Agents never push. The maintainer reviews local `main` and publishes:

```shell
git log --oneline origin/main..main
git diff origin/main..main
git push origin main
```

**Branch naming for the maintainer:** use one stable branch name per harness
(`vskilo` for the Kilo/VS Code setup, `opcode` for opencode) and keep one
worktree per name. Because parameterless `just merge` refuses when several
worktrees exist, pass the branch explicitly: `just merge vskilo`.

## Shell Functions

```shell
pyinit [name] [--lib] [--cli] [--python X.Y] [--help]
act <envname>
jupyter-launcher [-n] [-c] [-p PORT] [dir]
jl  # alias for jupyter-launcher (default: -n -c)
cw / cw .
rlb
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| ModuleNotFoundError | `uv sync` or forgot `uv run` |
| Wrong Python version | `uv python pin 3.12` |
| direnv not activating | `direnv allow` |
| Mamba env conflicts | recreate env |
| Lint errors | `just fix` |
| `just check` fails on documentation | run `just docs` and commit the regenerated page |
| bump-my-version fails | clean git state required |
| Jupyter MCP server fails with "Address already in use" | The `jupyter_server_mcp` extension defaults to port **3001**, which may conflict with other services. Create `~/.jupyter/jupyter_config.py` with `c.MCPExtensionApp.mcp_port = <free-port>`. Example: `echo "c.MCPExtensionApp.mcp_port = 3003" > ~/.jupyter/jupyter_config.py` and restart Jupyter. |
| `jupyter_server_mcp` error: "No module named 'pydantic_settings'" | After recreating the Mamba env, `mcp>=2` gets installed which is incompatible with `fastmcp`. Add `"fastmcp>=3"` to the `uv pip install` line (pins `mcp==1.x`). See `eco-systems/02-python.md` line 117. |
| JupyterLab 404 polling: `/api/collaboration/room/JupyterLab:globalAwareness` every 3s | Even with RTC extensions disabled, the JupyterLab client still polls this endpoint. Fix: `mkdir -p ~/.jupyter/lab/user-settings/\@jupyterlab/collaboration-extension/ && echo '{"collaborative":false,"enableDocs":false}' > ~/.jupyter/lab/user-settings/\@jupyterlab/collaboration-extension/plugin.jupyterlab-settings` (then reload browser tab). |
