# Projekt-Features: lupysta

Stand: 18. Oktober 2026  
Bezug: [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md), [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md), [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md)

## 1. Grundsatz

Ein Feature ist eine abgrenzbare, von `pyinit` erzeugte Projektfunktion. Jedes Feature hat:

- einen **Einstiegspunkt** (Rezept oder dokumentierter Befehl),
- eine **Dateimenge**, die ihm gehört,
- **Abhängigkeiten** (Laufzeit, Dev-Gruppen, Systemwerkzeuge),
- eine **pytest-Konfiguration** (Marker, Collection-Gate, `addopts`),
- **Dokumentation** im Projekt und im Skill,
- **Ignore- und Clean-Regeln** für seine Artefakte,
- eine **Entfernungsanleitung**.

Ein Feature darf fehlen. Fehlende Features werden nicht automatisch wiederhergestellt. Die Projekt-`AGENTS.md` bleibt die einzige Quelle für die Feature-Auswahl; der installierte Skill beschreibt den Workflow, nicht den Projektzustand.

## 2. Feature-Katalog

| Feature | Standard | Einstieg | Wichtige Dateien | Abhängigkeiten |
| --- | --- | --- | --- | --- |
| `core` | immer | `just check` | Quellcode, `Justfile`, `AGENTS.md`, `pyproject.toml`, CI, Smoke-Test | `uv`, `ruff`, `pytest`, `basedpyright`, `coverage` |
| `design-refactor` | vorbereitet | `just rate-design`, `just refactor` | `scripts/design_report.py` (schlank), `.refactor/` | `radon`, `vulture` |
| `e2e` | vorbereitet, entfernbar | `just test-e2e` | `tests/e2e/`, `scripts/test-e2e.sh`, `Dockerfile`, `README.md` | `pytest`, Docker, Gruppe `e2e` |
| `agent` | vorbereitet, entfernbar | `just test-agent` | `tests/agent/`, Harness, `agent.toml`, `README.md` | `pytest`, `httpx`, ggf. MCP, Gruppe `agent` |
| `docs-lib` | nur Bibliothek | `just docs`, `just docs-check` | `mkdocs.yml`, `docs/` | `mkdocs-material`, `mkdocstrings` |
| `docs-cli` | App mit `--cli` | `just docs`, `just docs-check` | `scripts/gen_cli_docs.py`, `docs/reference/cli.md` | `typer` |
| `setup-zsh` | optional, nur Doku | Anleitung | `references/lupysta-setup.md` | Antidote, Powerlevel10k |
| `setup-mamba` | optional, nur Doku | Anleitung | `references/lupysta-setup.md` | Mamba, Jupyter |

Die Standardauswahl hängt an D13. `docs-lib` und `docs-cli` schließen sich nicht aus, sind aber projekttypspezifisch: `docs-lib` gilt für Bibliotheken, `docs-cli` für Apps mit `--cli`.

## 3. Feature-Tabelle in `AGENTS.md`

Die erzeugte `AGENTS.md` enthält eine Tabelle mit diesen Spalten:

| Spalte | Inhalt |
| --- | --- |
| `Feature` | Feature-ID aus dem Katalog |
| `Status` | `available`, `removed`, `not-generated`, `legacy` |
| `Entry point` | Konkreter Befehl im Projekt, z. B. `just test-e2e` |
| `Recipe` | Rezeptname im Skill, der das Feature beschreibt |
| `Verify` | Befehl zur Prüfung, z. B. `just test-e2e --help` oder `just lupysta-status` |

Statuswerte:

- **`available`** – erzeugt und nutzbar.
- **`removed`** – bewusst entfernt; bleibt sichtbar, damit Agenten es nicht ungefragt wiederherstellen.
- **`not-generated`** – für diesen Projekttyp nicht vorgesehen (z. B. `docs-cli` in einer Bibliothek).
- **`legacy`** – aus einer älteren lupysta-Version übernommen, ohne Garantie auf neue Rezepte.

Zusätzlich nennt die `AGENTS.md` `lupysta_version` und `contract_version` (D10). Der vollständige Workflow-Vertrag bleibt im Skill und wird nicht in jede `AGENTS.md` kopiert (Review R6).

## 4. Gemeinsame Infrastruktur und Besitz

| Datei | Besitz | Regel beim Entfernen |
| --- | --- | --- |
| `tests/conftest.py` | gemeinsam von `e2e` und `agent` | Nur entfernen, wenn keines der beiden Features bleibt. Enthält das Collection-Gate für beide Suiten. |
| `pyproject.toml` | `core`, mit Beiträgen je Feature | Nur die eigenen Marker, `addopts`-Anteile und Dependency-Groups entfernen; `core`-Einträge bleiben. |
| `Justfile` | `core`, mit einem Rezeptblock je Feature | Jedes Feature besitzt seinen Block; Blöcke sind durch Kommentarüberschriften getrennt und einzeln entfernbar. |
| `.gitignore` | `core`, mit Artefaktzeilen je Feature | Nur die eigenen Artefaktzeilen entfernen. |
| `just clean` | `core`, mit einem Clean-Abschnitt je Feature | Feature-Artefakte einzeln entfernen; `.refactor/baselines/` bleibt erhalten (D5, R8). |
| `README.md` | `core` | Feature-Abschnitt und Tabellenzeile entfernen. |
| `CONTRIBUTING.md` | `core` | Feature-spezifische Hinweise entfernen. |
| `AGENTS.md` | `core` | Status des Features auf `removed` setzen, Einstiegspunkt und Rezept entfernen. |

Gemeinsame Dateien dürfen nie pauschal gelöscht werden. Jedes Feature trägt in `AGENTS.md` seinen Besitzanteil; die Entfernungsanleitung nennt die betroffenen Stellen.

## 5. Entfernungscheckliste

1. Feature-Dateien und -Verzeichnisse löschen.
2. Rezeptblock im `Justfile` entfernen.
3. Dependency-Group und zugehörige Lock-Einträge entfernen (`uv lock` neu erzeugen).
4. Marker, `addopts`-Anteil und Collection-Gate bereinigen; bei `e2e`/`agent` prüfen, ob `tests/conftest.py` noch gebraucht wird.
5. `.gitignore`-Zeilen und `just clean`-Abschnitt entfernen.
6. Doku in `README.md`, `CONTRIBUTING.md` und ggf. `docs/` entfernen.
7. Feature in `AGENTS.md` auf `removed` setzen.
8. `just check` ausführen.
9. `just lupysta-status` ausführen.
10. Verbleibende Suiten prüfen: `just test`, ggf. `just test-e2e`, ggf. `just test-agent`.

Nach der Entfernung gilt: Der Standardlauf bleibt grün, ohne Docker oder Provider. Agenten erhalten keine Anleitung für ein fehlendes Feature.

## 6. `just lupysta-status`

Der Befehl liest die Feature-Tabelle aus `AGENTS.md` und prüft, ob die deklarierten Einstiegspunkte vorhanden sind:

- Rezept im `Justfile` vorhanden,
- genannte Dateien vorhanden,
- Dependency-Group in `pyproject.toml` vorhanden (nur bei `available`),
- Statuswert gültig.

Der Befehl **ändert nichts**, erkennt keine Features automatisch und repariert nichts. Ausgabe: eine Zeile je Feature mit `ok`, `missing` oder `stale`. Exitcode 0, wenn alle `available`-Features stimmen; 1 bei Abweichungen. `removed`, `not-generated` und `legacy` erzeugen keine Fehler.

## 7. Standardauswahl und Flags

Die Standardauswahl ist offen (D13). Vorgesehene Flags:

| Flag | Wirkung |
| --- | --- |
| `--no-e2e` | `e2e` nicht erzeugen; Status `not-generated`. |
| `--no-agent` | `agent` nicht erzeugen; Status `not-generated`. |
| `--no-refactor` | `design-refactor` nicht erzeugen; Status `not-generated`. |
| `--minimal` | nur `core` (und projekttypspezifische Docs, falls zutreffend). |

Flags sind unabhängig kombinierbar, sofern sie sich nicht widersprechen. Jede Kombination wird getestet (siehe [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md)). Ohne Flag gilt die in D13 festgelegte Auswahl.

## 8. Tests

Für das Feature-Modell werden repräsentative Projekte erzeugt und geprüft:

- vollständiges Projekt mit allen Features,
- Projekt mit `--no-e2e`,
- Projekt mit `--no-agent`,
- Projekt mit `--no-refactor`,
- Projekt mit `--minimal`,
- Projekt, in dem ein Feature nachträglich nach Checkliste entfernt wurde.

Erwartung je Fall:

- `just check` bleibt grün.
- `just lupysta-status` meldet keine Abweichung.
- Entfernte Features werden in `AGENTS.md` als `removed` geführt.
- `just test` sammelt keine E2E- oder Agent-Tests ein.
- Die Doku nennt keine Befehle für fehlende Features.

Die Tests laufen in CI und sind Teil der Etappe-6-Abnahme.
