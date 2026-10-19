# End-to-end testing (optional)

This is an **optional pattern**, not part of the `pyinit` scaffold. Most
projects are covered by unit tests that mock external tools. Add real
end-to-end tests only when you must exercise the installed command against real
external tools, the filesystem, or package managers — and never on a
workstation.

The pattern below is the one used by the reference project (LION); adapt the
names to your project.

## When to use it

- The behavior depends on a real external tool (`apt`, `dpkg`, `lspci`, …) or on
  operating-system state that is unsafe or impossible to mock faithfully.
- You want to verify the packaged/installed executable, not the imported source.
- A container can isolate every side effect so the host stays untouched.

If a mocked test can prove the behavior, prefer the mocked test. End-to-end tests
are slower, need Docker and network access, and are harder to keep deterministic.

## Shape of the setup

Keep the opt-in machinery out of the default test run:

```text
scripts/test-e2e.sh          # runner: build image, run container, collect artifacts
tests/e2e/
├── conftest.py              # guards + shared fixtures
├── Dockerfile               # the isolated test image
└── test_*.py                # scenarios, each marked with pytest.mark.e2e
```

Register the marker and exclude it by default so `just test` and `just check`
never run the container:

```toml
[tool.pytest.ini_options]
markers = ["e2e: end-to-end tests that run in a disposable Docker container"]
addopts = ["-ra", "-q", "-m", "not e2e"]
```

Add a dedicated recipe instead of folding E2E into `just check`:

```make
# Real CLI/package lifecycle in a disposable Docker container
test-e2e:
    bash scripts/test-e2e.sh
```

## Isolation rules

- **Disposable container, no host mounts.** Run the tests in a fresh container
  created from a project image; nothing outside the container may change.
- **No privileged mode, no host network** unless a scenario explicitly needs a
  build-time download. Do not enable runtime networking to fix a failure.
- **Temporary writable state.** Use a temporary `HOME`, `XDG_*` and work
  directory inside the container so the developer's environment is never read.
- **Explicit opt-in.** The runner sets a marker environment variable (for
  example `APP_E2E_CONTAINER=1`). The shared `conftest.py` skips the suite
  unless that marker is present, so a stray `pytest` cannot mutate a real
  system.
- **Root only inside the container.** If a scenario needs package mutation, it
  runs as root in the throwaway container and switches to an unprivileged user
  for the rest. The project itself never requires root.

## Runner responsibilities

A good `scripts/test-e2e.sh`:

1. Creates a unique `e2e-artifacts.*` directory (gitignored) for diagnostics.
2. Builds the image from the `Dockerfile` and the lock file (BuildKit for cache
   mounts), installing dependencies before the source so code-only changes
   rebuild quickly.
3. Starts a fresh container with the opt-in marker and no host mounts.
4. Runs `pytest tests/e2e -m e2e` with a JUnit report under the artifacts
   directory.
5. Copies the artifacts to the host and removes the container on exit, even
   after failures.

## Adding a scenario

- Mark the module with `pytestmark = pytest.mark.e2e`; the shared `conftest.py`
  applies the guard and provides the subprocess and environment fixtures.
- Invoke the **installed** command as a subprocess, not the imported module.
- Keep fixtures self-contained and order-independent; use synthetic secrets.
- Assert exit codes, output, persisted state and permissions — and that read-only
  commands write nothing.
- Record each subprocess command, exit code and output under the artifacts
  directory so failures are diagnosable.
- Report the exact commands you ran and any coverage limits. Run the ordinary
  quality gate first, then the E2E runner.

## Further reading

- [`../SKILL.md`](../SKILL.md) — the stack workflow and the `just` recipes.
- [`../../../luxuspythonstack.md`](../../../luxuspythonstack.md) — full stack documentation.
