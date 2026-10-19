> **Deutsche Arbeitsfassung.** Übersetzung von [daily-commands.md](daily-commands.md). Sie ist ab jetzt die gemeinsame Arbeitsgrundlage; die englische Fassung wird erst nach Abschluss der deutschen Fassung nachgezogen.

# Luxus Python Stack — Tägliche Befehle

Schnellreferenz für häufige Operationen. Für die vollständige Dokumentation siehe [`luxuspythonstack.md`](../../../luxuspythonstack.md).

## Projektinitialisierung

```shell
pyinit                          # App im aktuellen Verzeichnis
pyinit my-project               # App in neuem Verzeichnis
pyinit my-lib --lib             # Bibliothek in neuem Verzeichnis
pyinit my-app --cli             # App mit generierter Typer-CLI-Referenz
pyinit my-project --python 3.11 # Python-Version angeben (Standard: 3.12)
pyinit --help                   # Nutzung anzeigen (keine Nebenwirkungen)
```

`pyinit` initialisiert nur leere oder neue Verzeichnisse; es ist kein Updater.
`--cli` ist nur für Apps (`pyinit --lib --cli` wird abgelehnt). Nach der
Initialisierung einmal `direnv allow` ausführen, um die `.venv` automatisch zu
aktivieren.

## Umgebungsverwaltung

```shell
# Mamba (Stufe 1 — Data Science)
act <envname>             # aktivieren + in ~/.startenv speichern
mamba activate <envname>  # aktivieren ohne Speichern
mamba deactivate          # deaktivieren
jupyter-launcher [folder]         # Jupyter Lab (mit aktiviertem Token)
jupyter-launcher -n [folder]      # Jupyter Lab ohne Token (unsicher)
jupyter-launcher -c [folder]      # Jupyter Lab mit Colab-Origin
jupyter-launcher -n -c [folder]   # ohne Token + Colab
jl                                # Alias: jupyter-launcher -n -c

# UV/direnv (Stufe 2 — Projekte)
direnv allow   # .envrc erlauben (einmal pro Projekt)
uv sync        # nach git pull / Abhängigkeitsänderung synchronisieren
```

## Abhängigkeiten

```shell
uv add <package>          # Laufzeitabhängigkeit
uv add --dev <package>    # Abhängigkeit nur für die Entwicklung
uv remove <package>       # entfernen
uv sync                   # .venv mit der Lock-Datei synchronisieren
uv pip install <package>  # nur Stufe 1
```

## Code ausführen

```shell
just run                            # installierten Befehl ausführen (Apps)
uv run python -m <pkg>.main         # garantiert synchron (CI/Skripte)
uvx <tool> <args>                   # Werkzeug kurzzeitig ausführen
```

## Tests

```shell
just test                       # alle Tests mit Coverage
pytest -v                       # ausführlich
pytest -k "test_name" -s        # bestimmter Test, stdout anzeigen
```

Für opt-in End-to-End-Tests mit echten Werkzeugen in einem Wegwerf-Docker-Container
siehe [`e2e-testing_de.md`](e2e-testing_de.md) (englische Fassung:
[`e2e-testing.md`](e2e-testing.md)).

## Codequalität

```shell
just lint         # ruff + basedpyright
just typecheck    # nur Typprüfung
just check        # vollständiges Gate (Lint + Typecheck + Docs + Tests)
just fix          # automatische Korrektur
just clean        # Caches und generierte Artefakte entfernen
just audit        # Sicherheit: pip-audit
```

## Dokumentation

Wird nur für Bibliotheken und Apps erzeugt, die mit `--cli` angelegt wurden:

```shell
just docs         # MkDocs bauen (Lib) / docs/reference/cli.md neu erzeugen (App --cli)
just docs-check   # prüfen, ob die generierte Dokumentation aktuell ist (Teil von `just check`)
just docs-serve   # Live-Vorschau (nur Bibliotheken)
```

Niemals eine generierte Seite von Hand bearbeiten; `just docs` ausführen und das
Ergebnis committen.

## Refactoring

```shell
just rate-design  # Design-Metriken → .refactor/ (nur Bericht)
just refactor     # Refactoring-Sitzung vorbereiten (Preflight, Baseline, Branch)
```

`rate-design` schlägt bei Befunden niemals fehl; `refactor` verweigert einen
nicht sauberen Arbeitsbaum, führt `just check` aus, schreibt die Baseline und
erstellt einen Branch `refactor/<slug>`, ohne Code zu ändern. Danach dem
[Refactoring-Skill](../../refactoring/SKILL.md) folgen: zuerst messen, ein Rezept
pro Schritt, `just check` vorher und nachher grün.

## Version & Release

```shell
just bump patch   # 0.1.0 → 0.1.1
just bump minor   # 0.1.1 → 0.2.0
just bump major   # 0.2.0 → 1.0.0
git push origin main --tags
```

## Git-Workflow

Auf einem Branch arbeiten (`feat/...`, `fix/...`, `docs/...`), bevorzugt in einem
separaten Worktree. `just check` vor dem Mergen grün halten:

```shell
git worktree add ../<project>-feature -b feat/my-feature
# ... Änderungen vornehmen ...
just check
git add -A && git commit -m "feat: ..."

# Nach git pull: uv sync
```

Vom `main`-Checkout aus den Branch lokal mergen (nur Fast-Forward):

```shell
just merge feat/my-feature   # expliziter Branch
just merge                   # oder den Branch in einem anderen Worktree automatisch erkennen
```

`just merge` zieht `main` per Fast-Forward auf den Branch und verweigert den
Vorgang, wenn `main` divergiert ist; danach entfernt es den Worktree des Branches
und löscht den Branch. Ohne Argument erkennt es den in einem anderen Worktree
ausgecheckten Branch, sonst den einzigen lokalen Branch, der nicht in `main`
gemergt ist, und verweigert den Vorgang, wenn keiner oder mehrere passen.
Es pusht niemals.

Bei Divergenz auf `main` rebasen, `just check` erneut ausführen und wieder mergen:

```shell
git rebase main
just check
just merge feat/my-feature
```

Agenten pushen niemals. Der Maintainer prüft lokales `main` und veröffentlicht:

```shell
git log --oneline origin/main..main
git diff origin/main..main
git push origin main
```

**Branch-Namen für den Maintainer:** pro Harness einen stabilen Branch-Namen
verwenden (`vskilo` für das Kilo/VS-Code-Setup, `opcode` für opencode) und einen
Worktree pro Name behalten. Weil parameterloses `just merge` sich weigert, wenn
mehrere Worktrees existieren, den Branch explizit angeben: `just merge vskilo`.

## Shell-Funktionen

```shell
pyinit [name] [--lib] [--cli] [--python X.Y] [--help]
act <envname>
jupyter-launcher [-n] [-c] [-p PORT] [dir]
jl  # Alias für jupyter-launcher (Standard: -n -c)
cw / cw .
rlb
```

## Fehlerbehebung

| Problem | Lösung |
|---------|----------|
| ModuleNotFoundError | `uv sync` oder `uv run` vergessen |
| Falsche Python-Version | `uv python pin 3.12` |
| direnv aktiviert nicht | `direnv allow` |
| Mamba-Umgebungskonflikte | Umgebung neu erstellen |
| Lint-Fehler | `just fix` |
| `just check` schlägt bei der Dokumentation fehl | `just docs` ausführen und die neu erzeugte Seite committen |
| bump-my-version schlägt fehl | sauberer Git-Zustand erforderlich |
| Jupyter-MCP-Server schlägt mit "Address already in use" fehl | Die Erweiterung `jupyter_server_mcp` verwendet standardmäßig Port **3001**, der mit anderen Diensten kollidieren kann. `~/.jupyter/jupyter_config.py` mit `c.MCPExtensionApp.mcp_port = <freier-port>` anlegen. Beispiel: `echo "c.MCPExtensionApp.mcp_port = 3003" > ~/.jupyter/jupyter_config.py` und Jupyter neu starten. |
| `jupyter_server_mcp`-Fehler: "No module named 'pydantic_settings'" | Nach dem Neuerstellen der Mamba-Umgebung wird `mcp>=2` installiert, was mit `fastmcp` inkompatibel ist. `"fastmcp>=3"` zur `uv pip install`-Zeile hinzufügen (fixiert `mcp==1.x`). Siehe `eco-systems/02-python.md` Zeile 117. |
| JupyterLab 404-Polling: `/api/collaboration/room/JupyterLab:globalAwareness` alle 3 s | Selbst bei deaktivierten RTC-Erweiterungen fragt der JupyterLab-Client diesen Endpunkt weiter ab. Lösung: `mkdir -p ~/.jupyter/lab/user-settings/\@jupyterlab/collaboration-extension/ && echo '{"collaborative":false,"enableDocs":false}' > ~/.jupyter/lab/user-settings/\@jupyterlab/collaboration-extension/plugin.jupyterlab-settings` (danach den Browser-Tab neu laden). |
