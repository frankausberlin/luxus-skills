---
name: luxuspythonstack
description: >
  Expertenwissen zum Luxurious Python Stack — ein fünfstufiger
  Entwicklungsworkflow für Python-Projekte mit UV, direnv, ruff, basedpyright
  und KI-Agenten-Integration. Dieser Skill sollte PROAKTIV eingesetzt werden,
  wenn an Python-Projekten gearbeitet wird, die mit pyinit initialisiert wurden,
  wenn der Benutzer Projektgerüste, Umgebungsverwaltung, Automatisierung des
  Qualitätsgates, Versionsanhebungen oder Anleitung zum Arbeitsablauf von
  KI-Agenten-Sitzungen benötigt.
version: 0.4.0
license: MIT
platforms: [linux, darwin]
metadata:
  tags: [python, uv, direnv, ruff, basedpyright, quality-gate, pyinit, ai-agents]
  related_skills: [refactoring]
---

> **Deutsche Arbeitsfassung.** Übersetzung von [SKILL.md](SKILL.md). Sie ist ab jetzt die gemeinsame Arbeitsgrundlage; die englische Fassung wird erst nach Abschluss der deutschen Fassung nachgezogen.

# Luxurious Python Stack

## Überblick

Dieser Skill vermittelt Expertenwissen zum Luxurious Python Stack — einem fünfstufigen Entwicklungsworkflow für flexible, konfliktfreie Verwaltung von Python-Umgebungen. Der Stack trennt Zuständigkeiten in verschiedene Stufen, jede mit eigenen Werkzeugen und Zwecken.

## Installationsmodell

Der Stack wird über symbolische Links installiert, nicht durch Kopieren von Dateien. Dadurch bleibt der Repository-Checkout die einzige Quelle der Wahrheit und Aktualisierungen sind per `git pull` möglich. Die maßgebliche Installationsanleitung liegt im Projektstamm: [`luxuspythonstack.md`](../../luxuspythonstack.md).

Agenten sollten auf die maßgebliche Anleitung verweisen, statt vollständige Installationsbefehle zu duplizieren. Die wichtigsten Regeln:

- [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh) mit einem vom Benutzer gewählten zweistelligen Präfix für die Ladereihenfolge in die shlib-Einrichtung des Benutzers verlinken.
- Den Ordner `skills/luxuspythonstack` in jedes agentenspezifische Skill-Verzeichnis verlinken, in dem dieser Stack verfügbar sein soll.
- Kopierende Installationsanweisungen vermeiden.

## Wartungs- und Aktualisierungsrichtlinie

- Diesen Repository-Checkout als einzige bearbeitbare Quelle der Wahrheit behandeln.
- Keine Dateien in verlinkten Agent-Skill-Verzeichnissen patchen; stattdessen die Repository-Dateien bearbeiten.
- Symlinks bei der Fehlersuche nicht durch kopierte Dateien ersetzen.
- Installierte Integrationen mit `git pull` im Repository-Checkout aktualisieren und danach die Shell neu laden, wenn sich Shell-Funktionen geändert haben.
- Detaillierte Installationsschritte in [`luxuspythonstack.md`](../../luxuspythonstack.md) belassen; diese Skill-Datei auf Agentenverhalten und Betriebsregeln fokussieren.

## Fünfstufige Architektur

| Stufe | Name | Werkzeug | Aktivierung |
|-------|------|------|------------|
| 0 | Global / System | `/usr/bin/python` | Immer aktiv (Fallback) |
| 1 | Mamba / Jupyter | Mamba + UV | `act <envname>` oder Start über `~/.startenv` |
| 2 | Projekte | UV + direnv | Automatisch via `direnv`, wenn `.venv` existiert |
| 3 | CI / CD | GitHub Actions + UV | Bei Push/PR oder manueller Auslösung |
| 4 | KI-Agenten | AGENTS.md + SESSION.md | Start der Agenten-Sitzung |

**Grundprinzip**: Die Stufen sind isoliert. Stufe 2 (.venv) überschreibt Stufe 1 (Mamba) in jedem Verzeichnis mit einem `.venv`-Ordner über `direnv`.

## Entscheidungsbaum

```
Neu in einem Projektverzeichnis?
  → .venv vorhanden? → Bereits auf Stufe 2 (direnv aktiviert automatisch)
  → Kein .venv?      → pyinit ausführen (siehe ../../scripts/luxuspythonstack.sh) oder uv init

Paket installieren?
  → In einem Projekt (.venv aktiv): uv add <package>
  → In Data-Science-Umgebung:       uv pip install <package>
  → Systemweit:                     vermeiden (virtuelle Umgebungen nutzen)

Code ausführen?
  → Im Terminal mit direnv:  python src/...  (Umgebung automatisch aktiv)
  → In Skripten/CI:          uv run python src/...  (garantiert synchron)

Veröffentlichen?
  → just bump [patch|minor|major]
  → git push origin main --tags
```

## Zentrale Workflows

### 1. Ein neues Python-Projekt initialisieren

Die Shell-Funktion `pyinit` verwenden (definiert in [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh)):

```shell
pyinit                              # App im aktuellen Verzeichnis
pyinit my-project                   # App in neuem Verzeichnis
pyinit my-lib --lib                 # Bibliothek in neuem Verzeichnis
pyinit my-app --cli                 # App mit generierter Typer-CLI-Referenz
pyinit my-project --python 3.11     # Python-Version angeben (Standard: 3.12)
pyinit --help                       # Nutzung anzeigen (keine Nebenwirkungen)
```

`pyinit` initialisiert nur leere oder neue Verzeichnisse; es ist kein Updater.
`--cli` ist nur für Apps: `pyinit --lib --cli` wird abgelehnt.

Das Skript erzeugt:
- `src/<package>/` mit `__init__.py`, `main.py` (App) oder `py.typed` (Lib)
- `tests/test_smoke.py` (ein echter Smoke-Test, kein `assert True`)
- `pyproject.toml` mit ruff, basedpyright, pytest, einem 90-%-Coverage-Gate und bump-my-version-Konfiguration
- `.venv/` virtuelle Umgebung (standardmäßig Python 3.12)
- `.python-version`, `.vscode/` (Einstellungen, Launch, Erweiterungen)
- `.envrc` für die automatische Aktivierung durch direnv (mit Schutz gegen Conda-Deaktivierung)
- `.gitignore` (lokale Vorlage, keine Netzwerkanfrage)
- `.editorconfig`, `Justfile`, `.pre-commit-config.yaml`
- `.github/workflows/ci.yml` (immer) und `release.yml` (nur Bibliotheken)
- `AGENTS.md`, `SESSION.md`, `JOURNAL.md`, `README.md`, `CONTRIBUTING.md`, `LICENSE`
- **Nur Bibliotheksprojekte:** `mkdocs.yml` + `docs/` mit Material-Theme und mkdocstrings; `just docs-check` (`mkdocs build --strict`) ist Teil von `just check`
- **Nur App `--cli`:** `src/<package>/cli.py` (Typer-App), `scripts/gen_cli_docs.py` und eine generierte `docs/reference/cli.md`; `just docs-check` ist Teil von `just check`

Nach dem Ausführen von pyinit einmal `direnv allow` ausführen (ein manueller Schritt), um die `.venv` automatisch zu aktivieren.

### 2. Umgebungsverwaltung

**Mamba-Umgebung (Stufe 1)**:
```shell
act ds12          # aktiviert und speichert in ~/.startenv
mamba activate ds12  # ohne Speichern
```

**Projektumgebung (Stufe 2)**:
```shell
# direnv übernimmt die Aktivierung automatisch beim Betreten des Verzeichnisses
# Falls nach git pull Modulfehler auftreten:
uv sync
```

**Mamba-Umgebung neu erstellen**:
```shell
py=3.12 && ENV_NAME="ds${py: -2}"
mamba deactivate
mamba remove -y -n $ENV_NAME --all 2>/dev/null
mamba create -y -n $ENV_NAME python=$py <packages...>
mamba activate $ENV_NAME
```

### 3. Tägliche Entwicklungsbefehle

```shell
uv add <package>          # Laufzeitabhängigkeit
uv add --dev <package>    # Abhängigkeit nur für die Entwicklung
uv sync                   # Umgebung synchronisieren

just run                  # Hauptskript ausführen
just test                 # pytest mit Coverage
just lint                 # ruff + basedpyright
just typecheck            # nur Typprüfung
just check                # vollständiges Qualitätsgate (Lint + Typecheck + Docs + Tests)
just fix                  # automatische Korrektur
just clean                # Caches und generierte Artefakte entfernen
just audit                # Sicherheitsscan
just rate-design          # Design-Metriken → .refactor/ (nur Bericht)
just refactor             # Refactoring-Sitzung vorbereiten (Preflight, Baseline, Branch)
just merge                # main per Fast-Forward auf einen Branch ziehen und dessen Worktree/Branch aufräumen (kein Push)
```

Wenn Dokumentation generiert wird (`--lib` oder App `--cli`):

```shell
just docs                 # MkDocs bauen (Lib) / docs/reference/cli.md neu erzeugen (App --cli)
just docs-check           # prüfen, ob die generierte Dokumentation aktuell ist (Teil von `just check`)
just docs-serve           # Live-Vorschau (nur Bibliotheken)
```

### 4. Release & Deployment

Versionen niemals manuell bearbeiten oder Tags erstellen. Immer `bump-my-version` verwenden:

```shell
just bump patch           # 0.2.0 → 0.2.1
just bump minor           # 0.2.1 → 0.3.0
just bump major           # 0.3.0 → 1.0.0
git push origin main --tags
```

### 5. Arbeitsablauf für KI-Agenten-Sitzungen (Stufe 4)

KI-Agenten arbeiten über MCP-Server (Websuche, GitHub, Docker, Browser-Automatisierung usw.), die von MCPHub verwaltet werden. Die Server-Einrichtung ist im Abschnitt [AI-Stack](../../ai-stack/) dokumentiert — nur laden, wenn die Agenten-Infrastruktur konfiguriert wird.

**Start:**
1. `AGENTS.md` für den Projektkontext lesen
2. `SESSION.md` lesen, falls vorhanden
3. `uv sync` und `git status` ausführen
4. Arbeit beginnen

**Ende:**
1. `just check` ausführen — muss grün sein
2. `SESSION.md` mit dem aktuellen Stand **überschreiben** (flüchtig)
3. Datierter Eintrag an `JOURNAL.md` **anhängen** (Historie, niemals überschreiben)
4. Alle Änderungen auf dem Branch committen
5. Vom Haupt-Checkout aus lokal mit `just merge [branch]` mergen (nur Fast-Forward; entfernt den Worktree des Branches und löscht den Branch) — niemals pushen

## Wichtige Shell-Funktionen

Diese Shell-Helfer liegen in verschiedenen Dateien des Repositorys:

| Funktion | Verwendung | Definiert in | Beschreibung |
|----------|-------|------------|-------------|
| `pyinit` | `pyinit [name] [--lib] [--cli] [--python X.Y] [--help]` | [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh) | Ein neues Python-Projekt erstellen |
| `jupyter-launcher` | `jupyter-launcher [-n] [-c] [-p PORT] [folder]` | [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh) | Jupyter Lab starten |
| `jl` | `jl` | [`luxuspythonstack.sh`](../../scripts/luxuspythonstack.sh) | Alias für `jupyter-launcher` (Standard: `-n -c`) |
| `act` | `act <envname>` | [`eco-systems/02-python.md`](../../eco-systems/02-python.md) | Mamba-Umgebung aktivieren + in `~/.startenv` speichern |
| `cw` | `cw` / `cw .` | [`deenlupysta.sh`](../../scripts/deenlupysta.sh) | Zum Arbeitsordner springen / ihn setzen (globaler Zustand) |
| `rlb` | `rlb` | [`deenlupysta.sh`](../../scripts/deenlupysta.sh) | Shell neu laden (`exec zsh`) |

`jupyter-launcher` ist die maßgebliche Implementierung. `jl` ist absichtlich nur ein Alias, damit Benutzer persönliche Standardwerte vorkonfigurieren können, zum Beispiel `alias jl='jupyter-launcher -n -c'`.

## Wichtige Regeln

1. **`uv run` vs. direkte Ausführung**: In Skripten und CI immer `uv run` verwenden (synchronisiert automatisch). Im Terminal mit aktivem direnv ist direktes `python` in Ordnung.

2. **Niemals uv-Pakete mit Mamba vermischen**: Auf Stufe 1 steht `uv pip install` absichtlich nicht unter Mambas Kontrolle. Die Data-Science-Umgebung ist wegwerfbar und wird regelmäßig neu aufgebaut.

3. **Versionsanhebung**: Niemals `version =` in `pyproject.toml` oder `__init__.py` manuell bearbeiten. Niemals manuell `git tag` ausführen. Immer `just bump` verwenden.

4. **direnv**: Wenn die automatische Aktivierung nicht mehr funktioniert, `direnv allow` im Projektverzeichnis ausführen.

5. **Generierte Dokumentation**: Niemals `docs/reference/cli.md` oder andere generierte Seiten von Hand bearbeiten. `just docs` ausführen und das Ergebnis committen; `just check` schlägt fehl, wenn eine generierte Seite veraltet ist.

6. **Git-Workflow**: Auf einem Branch arbeiten (`feat/...`, `fix/...`, `docs/...`), `just check` grün halten und dann vom Haupt-Checkout aus lokal mit `just merge [branch]` mergen. Agenten pushen niemals; der Maintainer prüft lokales `main` und veröffentlicht mit `git push origin main`.

## Referenzdateien

Nach Bedarf laden:

- [`references/daily-commands_de.md`](references/daily-commands_de.md) — Vollständige Befehlsreferenz (deutsche Arbeitsfassung; englische Fassung: [`daily-commands.md`](references/daily-commands.md))
- [`references/e2e-testing_de.md`](references/e2e-testing_de.md) — Optionales Docker-isoliertes End-to-End-Testmuster (deutsche Arbeitsfassung; englische Fassung: [`e2e-testing.md`](references/e2e-testing.md))
- [`../refactoring/SKILL.md`](../refactoring/SKILL.md) — Sicherer, messbarer Refactoring-Workflow (`rate-design`/`refactor`) und Rezeptkatalog
- [`../../base-system/`](../../base-system/) — Vollständige Einrichtungsanleitungen für die Entwicklerumgebung
- [`../../ai-stack/`](../../ai-stack/) — Einrichtung der AI-Stack-Komponenten
- [`../../luxuspythonstack.md`](../../luxuspythonstack.md) — Vollständige Dokumentation und maßgeblicher `stack-contract-v1`-Export für generierte `AGENTS.md`-Dateien
