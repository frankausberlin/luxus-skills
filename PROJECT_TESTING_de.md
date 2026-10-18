# Projekt-Testing: lupysta

Stand: 18. Oktober 2026  
Bezug: [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md), [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md), [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md)

## Zweck

Dieses Dokument beschreibt die Teststrategie für zwei Ebenen:

1. das **Skill-Repository** selbst (Shell-Funktionen, Vorlagen, Links, Installation),
2. die von `pyinit` **erzeugten Projekte** (Standardtests, E2E, Agent, Feature-Entfernung).

Es legt außerdem fest, wie optionale Suiten von der normalen Collection getrennt bleiben und wie leere Suiten gemeldet werden.

## 1. Testebenen des Skill-Repositories

| Ebene | Inhalt | Laufzeit | Auslöser |
| --- | --- | --- | --- |
| **Schnell** | Argumente, Guards, Contract/Template, Hilfsfunktionen; keine Netzwerkzugriffe, keine Projektinitialisierung | Sekunden | jeder Push, lokal vor dem Commit |
| **Integration** | App, CLI-App, Bibliothek, Bash/Zsh, Symlink-Installation, `just check` im erzeugten Projekt | Minuten; Netzwerk und `uv` erforderlich | CI, vor Releases |
| **Plattform** | Linux als Referenz; macOS nur bei Zusage (D2) | Minuten bis Stunden | CI-Matrix, macOS nur bei Plattformzusage |
| **Optional/langsam** | E2E und Agent in erzeugten Projekten | Minuten; Docker bzw. Provider erforderlich | manuell, separater CI-Workflow |

Die schnelle Ebene ist die Pflichtstufe. Sie muss ohne `ignore/` und ohne das alte Repository laufen. Die Integrationsebene darf Netzwerk benötigen, muss aber reproduzierbar sein (D11).

## 2. CI für das Skill-Repository

Pflichtjobs:

- **Link-Checker** über alle Markdown-Dateien; keine toten internen Links, keine Pflichtverweise außerhalb des Repositories.
- **Shellcheck** für `lupysta/scripts/*.sh` und `tests/*.sh`.
- **Schnelle Tests** (`tests/test-pyinit.sh --fast`).
- **Integrationstests** (App, CLI-App, Bibliothek, Bash/Zsh, Symlink).
- **Fresh-Clone-Test**: Repository ohne `ignore/` auschecken, Skill verlinken, `source`, Projekt erzeugen, `just check`.

Zusätzlich:

- Kein Job darf `deen-lupysta` oder `ignore/` voraussetzen.
- Ein macOS-Job existiert nur, wenn D2 macOS zusagt.
- Optionaler, separat ausgelöster Workflow für E2E und Agent in erzeugten Projekten.

## 3. Generierte Projekte

### Standard (`core`)

- Smoke- und Unit-Tests aus `pyinit`.
- `just test` und `just check` sind grün.
- Coverage-Mindestwert 90 % für den Standardlauf.
- `just check` umfasst Ruff, Formatprüfung, basedpyright und Tests; bei Bibliotheken und CLI-Apps zusätzlich `just docs-check`.

### E2E (`e2e`)

- Testdateien unter `tests/e2e/`, Runner `scripts/test-e2e.sh`, Rezept `just test-e2e`.
- Docker erforderlich; keine Host-Mounts für veränderbare Benutzerdaten.
- Artefakte (JUnit, Logs) unter einem definierten Pfad, in `.gitignore` und `just clean` berücksichtigt.
- Kein Coverage-Mindestwert.

### Agent (`agent`)

- Testdateien unter `tests/agent/`, Rezept `just test-agent`.
- Provider-Preflight vor jedem Lauf; Report und Transkript werden geschrieben.
- Keine Providerzugriffe im normalen `pytest`, in `just test` oder `just check`.
- Kein Coverage-Mindestwert.

## 4. Collection-Trennung

Das Root-`tests/conftest.py` steuert die Collection beider optionaler Suiten:

```python
import os

collect_ignore = []
if os.environ.get("LUPYSTA_E2E") != "1":
    collect_ignore.append("e2e")
if os.environ.get("LUPYSTA_AGENT") != "1":
    collect_ignore.append("agent")
```

Die Pfade sind relativ zum Verzeichnis der `conftest.py`. Ausgeschlossen werden die **Verzeichnisse selbst**, nicht nur ihre Inhalte. Muster wie `e2e/*` genügen nicht: pytest betritt das Verzeichnis und lädt dessen `conftest.py`, bevor die Inhalte gefiltert werden. Ein optionaler Importfehler bricht dann den Standardlauf ab. Der Unterschied ist mit einem minimalen Projekt belegt (siehe `tests/test_collection_gate.py`); dort steht auch der Aufruf.

Regeln:

- `LUPYSTA_E2E=1` bzw. `LUPYSTA_AGENT=1` aktivieren die jeweilige Suite.
- Die dedizierten Runner setzen genau eine dieser Variablen und adressieren explizit nur ihre Suite (z. B. `pytest tests/e2e`).
- Marker `e2e` und `agent` gelten nur für die dedizierten Läufe und ergänzen das Gate, ersetzen es nicht.
- Keine optionalen Importe (`docker`, `httpx`, Provider-SDKs) auf Modulebene. Importe erfolgen in Fixtures oder innerhalb der Tests.
- Das Gate wirkt vor der Collection; `-m 'not e2e'` allein genügt nicht (Review R9).
- `tests/conftest.py` gehört beiden Suiten, bleibt frei von optionalen Abhängigkeiten und wird erst entfernt, wenn keine der beiden mehr vorhanden ist.
- Collection-Fehler einer ausdrücklich aktivierten Suite werden nicht verschluckt.

## 5. Null-Test-Semantik

| Zustand | Verhalten |
| --- | --- |
| Keine `test_*.py`-Dateien in der Suite | Klare Meldung, Exitcode 0, kein Docker, kein Provider. |
| Dateien vorhanden, Collection fehlerhaft | Fehler, Exitcode ungleich 0. |
| Dateien vorhanden, Ausführung fehlerhaft | Fehler, Exitcode ungleich 0. |
| `pytest`-Exitcode 5 | Nur im echten Leerfall tolerieren; bei vorhandenen Dateien als Fehler behandeln. |

Die Runner prüfen daher **zuerst** auf Testdateien und **danach** den Docker- bzw. Provider-Preflight. Ein fehlender Docker-Daemon bei leerer Suite ist kein Fehler.

## 6. E2E-Runner

Ablauf von `just test-e2e`:

1. Testdateien unter `tests/e2e/` prüfen; bei leerer Suite Meldung und Exitcode 0.
2. Docker-Preflight (Daemon erreichbar, Image baubar); Fehler mit klarer Meldung.
3. Image bauen.
4. Container ohne Host-Mounts für veränderbare Daten starten.
5. Tests ausführen, JUnit-Bericht und Logs einsammeln.
6. Container und temporäre Daten aufräumen, auch im Fehlerfall.

Dockerfile-Anforderungen:

- Projekt-Python-Version verwenden.
- `uv sync --locked --no-dev --group e2e`; keine Dev-Werkzeuge im Image.
- Unprivilegierter Testnutzer.
- `/artifacts` als Ablage für JUnit und Logs.
- Basis-Images nach Möglichkeit per Digest pinnen.

Ein separater CI-Workflow ist optional und wird nur für Projekte mit `e2e` erzeugt.

## 7. Agent-Runner

Ablauf von `just test-agent`:

1. Testdateien unter `tests/agent/` prüfen; bei leerer Suite Meldung und Exitcode 0.
2. Provider-Preflight: Endpoint, Modell, Zugangsdaten; fehlende Voraussetzungen führen zu einem lauten Fehler, nie zu einem stillen Skip.
3. Limits prüfen (Zeit, Schritte, Kostenrahmen).
4. Lauf ausführen, Report und Transkript schreiben.
5. Ergebnis melden; modellabhängige Qualität bleibt eine Ergänzung zum deterministischen Quality Gate.

Fake-Provider-Tests decken Limits, Fehlerpfade und Report-Schema ohne Live-Modell ab. Ein Live-Lauf wird manuell dokumentiert und ist kein Release-Gate. Der Report enthält mindestens Status, Szenario, Limitverletzungen und Verweis auf das Transkript; ein Preflight-Fehler wird als eigener Status (`preflight_error`) geführt.

## 8. Akzeptanzmatrix

Die Matrix verbindet Etappen und Abnahmekriterien. „rc“ ist der erwartete Exitcode. Die Matrix ist ein Zielbild: Noch nicht vorhandene Skripte und Rezepte sind als geplante Abnahme formuliert, nicht als bereits ausgeführte Nachweise. Bereits ausführbar ist derzeit `tests/test_collection_gate.py`.

### Etappe 1 – Eigenständiger Skill

| Prüfung | Befehl | rc |
| --- | --- | --- |
| Schnelle Tests | `bash tests/test-pyinit.sh --fast` | 0 |
| Vollständige Tests | `bash tests/test-pyinit.sh` | 0 |
| Shellcheck | `shellcheck lupysta/scripts/luxuspythonstack.sh tests/*.sh` | 0 |
| Link-Check | `bash scripts/check-links.sh` | 0 |
| Fresh Clone ohne `ignore/` | `bash tests/fresh-clone.sh` | 0 |
| Projekt erzeugen (App) | `pyinit /tmp/demo-app` | 0 |
| Projekt prüfen | `cd /tmp/demo-app && just check` | 0 |
| Nicht-leeres Ziel | `pyinit /tmp/non-empty` (mit Inhalt) | 1 |
| Ziel mit `.git` | `pyinit /tmp/cloned` (frischer Klon) | 0 |
| Fehlender Contract | `pyinit /tmp/x` mit entferntem `resources/pyinit/` | 1 |
| Initialisierung ohne Gate | `pyinit /tmp/demo --no-check` | 0 |

### Etappe 2 – Dokumentation, Setup, Feature-Modell

| Prüfung | Befehl | rc |
| --- | --- | --- |
| Links im Skill | `bash scripts/check-links.sh` | 0 |
| Feature-Tabelle vorhanden | `grep -q 'lupysta_version' /tmp/demo-app/AGENTS.md` | 0 |
| Statusbefehl grün | `cd /tmp/demo-app && just lupysta-status` | 0 |
| Statusbefehl erkennt Lücke | Rezept entfernen, dann `just lupysta-status` | 1 |
| Interne Links lösen sich auf | `bash scripts/check-links.sh` | 0 |
| Defekter relativer Link wird erkannt | `bash tests/test-check-links.sh` (Fixture) | 0 |
| Keine Pflichtdatei außerhalb des Checkouts | `bash tests/fresh-clone.sh` | 0 |
| Collection-Gate | `uvx --from pytest pytest tests/test_collection_gate.py` | 0 |

### Etappe 3 – Design/Refactor

| Prüfung | Befehl | rc |
| --- | --- | --- |
| Messung | `cd /tmp/demo-app && just rate-design` | 0 |
| Baseline speichern | `just rate-design save before` | 0 |
| Vergleich | `just rate-design compare before` | 0 |
| Refactor-Vorbereitung | `just refactor demo` | 0 |
| Refactor bei dirty tree | `touch src/change.py && just refactor demo2` | 1 |
| Clean erhält Baselines | `just clean && test -d .refactor/baselines` | 0 |

Session-Übergabe (geplant, D5; derzeit nicht implementiert):

| Fall | Erwartetes Verhalten |
| --- | --- |
| Worktree enthält `SESSION.md`/`JOURNAL.md`, sichere Übergabe nicht möglich | Git-Merge wird gemeldet; Worktree bleibt mit Pfad und Hinweis stehen. |
| Worktree enthält keine Session-Dateien | Worktree und Branch werden nach dem Merge entfernt. |
| Gleichnamige Dateien mit unterschiedlichem Inhalt | Dateien im Haupt-Checkout bleiben unverändert; Konflikt wird gemeldet, Worktree bleibt. |
| Schreibfehler bei einer späteren automatischen Übergabe | Fehler wird gemeldet; Worktree bleibt; kein Force-Remove. |

### Etappe 4 – E2E

| Prüfung | Befehl | rc |
| --- | --- | --- |
| Leere Suite ohne Docker | `just test-e2e` | 0 |
| Standardlauf sammelt nichts ein | `just test` | 0 |
| Ein Test im Container | Test ergänzen, `just test-e2e` | 0 |
| Fehlerhafter Test | Test ergänzen, der scheitert; `just test-e2e` | ≠ 0 |
| Preflight ohne Docker | Docker stoppen, Test vorhanden, `just test-e2e` | ≠ 0, klare Meldung |
| Artefakte | `test -d .artifacts/e2e` | 0 |

### Etappe 5 – Agent (nur mit Go/No-Go)

| Prüfung | Befehl | rc |
| --- | --- | --- |
| Leere Suite ohne Provider | `just test-agent` | 0 |
| Standardlauf sammelt nichts ein | `just test` | 0 |
| Fake-Provider: Limits | `LUPYSTA_AGENT=1 pytest tests/agent -q` | 0 |
| Fehlender Provider bei vorhandenem Test | `just test-agent` ohne Zugangsdaten | ≠ 0, `preflight_error` |
| Live-Lauf | dokumentierter manueller Lauf | 0 |

### Etappe 6 – Stabilisierung und Release

| Prüfung | Befehl | rc |
| --- | --- | --- |
| Feature-Entfernung | Projekt ohne `--no-e2e` erzeugen, `e2e` nach Checkliste entfernen, `just check` | 0 |
| Status nach Entfernung | `just lupysta-status` | 0 |
| Minimalprojekt | `pyinit /tmp/min --minimal && cd /tmp/min && just check` | 0 |
| Integrationsmatrix | `bash tests/integration.sh` | 0 |
| Release-Checkliste | dokumentierte Prüfung | 0 |

Die Matrix ist die gemeinsame Referenz für Roadmap und Umsetzung. Abweichungen werden zuerst hier korrigiert und dann in die Roadmap übernommen.
