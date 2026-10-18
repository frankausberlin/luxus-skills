# Projekt-Roadmap: lupysta

Stand: 18. Oktober 2026  
Status: abgestimmte Richtung; Etappe 1 ist noch nicht abnehmbar (siehe Review). Technische Details werden in der jeweiligen Etappe konkretisiert.

**Begleitdokumente**

| Dokument | Inhalt |
| --- | --- |
| [`PROJECT_REVIEW_de.md`](PROJECT_REVIEW_de.md) | Befunde R1–R15, Evidenz, Lean-MVP-Empfehlung |
| [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md) | Entscheidungslog D1–D15 mit Status |
| [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md) | Feature-Modell, Besitz, Entfernung, `just lupysta-status` |
| [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md) | Test- und CI-Strategie, Akzeptanzmatrix |
| [`PROJECTNOTE_de.md`](PROJECTNOTE_de.md) | Ursprüngliche Projektnotiz |

## Ziel

Im Repository `luxus-skills` entsteht mit `lupysta` ein eigenständiger, möglichst stabiler und universeller Skill für den Luxus Python Stack. Er bündelt Projektinitialisierung, Entwicklungsabläufe und verständliche Einrichtungsanleitungen für Menschen und Agenten.

Neue pyinit-Projekte erhalten neben den normalen Tests sofort nutzbare Grundgerüste für E2E- und Agent-Tests sowie die weiterentwickelten Werkzeuge für Designbewertung und Refactoring-Vorbereitung.

## Leitentscheidungen

- **Ein zentral installierter Skill:** Keine zusätzliche lupysta-Skill-Kopie im erzeugten Projekt. Dadurch entstehen keine konkurrierenden Skill-Versionen im Harness.
- **Zentraler Workflow, projektspezifische Auswahl:** Der installierte Skill beschreibt den gesamten Entwicklungs- und Auslieferungsworkflow. Die Projekt-`AGENTS.md` enthält lokale Regeln und die Auswahl der verfügbaren Features. Der Skill ist für die effiziente Arbeit durch Agenten vorgesehen; ausführbare Projektdateien werden von pyinit erzeugt.
- **Rezepte bei Bedarf laden:** `SKILL.md` enthält einen kurzen Gesamtüberblick und ein Verzeichnis der Rezepte. Agenten laden nur die für die Aufgabe und die verfügbaren Projektfeatures relevanten Detailanleitungen.
- **Projektzustand bestimmt die Verwendung:** Die Feature-Auswahl in `AGENTS.md` nennt Einstiegspunkte und stabile Rezeptnamen, keine absoluten Skill-Pfade. Vor Verwendung prüfen Agenten die tatsächlichen Skripte und Justfile-Rezepte; bei älteren Projekten werden neue Funktionen nicht vorausgesetzt.
- **Entfernbare Features:** Projekte dürfen generierte Features wieder entfernen. Dabei werden Feature-Auswahl, Konfigurationen und Abhängigkeiten mit angepasst. Andere Features und ihre Anleitungen bleiben nutzbar; fehlende Features werden nicht ungefragt wiederhergestellt.
- **Initialisierung statt Aktualisierung:** pyinit erzeugt neue Projekte. Bestehende Projekte und ihre angepassten Dateien werden nicht automatisch synchronisiert.
- **Nachvollziehbare Herkunft:** Die erzeugte `AGENTS.md` nennt `lupysta_version` und `contract_version`; daraus entsteht keine Versionsbindung.
- **Optionale Nutzung:** E2E und Agent werden grundsätzlich vorbereitet, bleiben aber außerhalb von `just test`, `just check` und der standardmäßig erzeugten CI.
- **Plattform (vorläufig):** Linux ist die Referenzplattform. macOS wird erst mit CI- und Testabdeckung zugesagt (D2, offen).
- **Lizenz (vorläufig):** Vor der ersten Veröffentlichung zu klären; Empfehlung Apache-2.0 mit `NOTICE` für MIT-Teile (D3, offen).
- **Session-Zustand:** Worktree-tauglich. `SESSION.md` und `JOURNAL.md` bleiben gitignored. Ein Merge darf keine lokalen Inhalte verlieren: Ohne nachweislich sichere Übergabe bleibt das Worktree stehen; eine automatische Zusammenführung ist nicht Teil des Pflichtumfangs (D5, vorgeschlagen).
- **Agent-Harness:** Minimaler Kern statt vollständiger Lion-Port; Go/No-Go nach Etappe 4 (D6, vorgeschlagen).
- **Feature-Modell:** Feature-Tabelle in `AGENTS.md` plus `just lupysta-status`; kein Rezeptmanager, keine automatische Feature-Erkennung (D8, vorgeschlagen).
- **Sprache:** Skill, Code und reguläre Dokumentation sind englisch. Projektnotiz, Roadmap, Review, Entscheidungen, Features und Testing sind deutsch (D15).
- **Bewusste Auslagerung:** Der bisherige Skill in `deen-lupysta` bleibt als alte Version bestehen. Die neue Entwicklung erfolgt in `luxus-skills`.

## Querschnittsaufgaben

Diese Aufgaben sind nicht optional und werden Etappe 1/2 zugeordnet:

| Aufgabe | Inhalt | Etappe | Bezug |
| --- | --- | --- | --- |
| **Portabilität** | GNU-spezifische Werkzeuge ersetzen oder Plattformzusage einschränken (`readlink -f`, `sed -i`, `mapfile`, `realpath -m`). | 1 | R2, D2 |
| **Transaktionale Initialisierung** | Vollständige Vorprüfung vor dem ersten Schreibzugriff; kein halbes Projekt; `.git`-only erlauben; `--no-check`. | 1 | R3 |
| **Tests und CI** | Alte `test-pyinit.sh` übernehmen, schnell/langsam trennen, CI einrichten. | 1 | R5, D14 |
| **Link-Checker und Shellcheck** | Portabler Link-Checker, Shellcheck für alle Shell-Dateien. | 1 | R5, R15 |
| **Lizenz** | `LICENSE` und ggf. `NOTICE`; Herkunft der übernommenen Teile. | 1 | R4, D3 |
| **Migration und Uninstall** | Alter Skill entlinken, nicht beide installieren, Uninstall dokumentieren. | 1 | R11, D12 |
| **Reproduzierbare Tool-Versionen** | Getestete Versionen von `uv`, `just`, `git`, `bash`, `direnv`/`layout_uv`; Netzwerk, Docker und Provider ausweisen. | 1/2 | R10, D11 |
| **Feature-Besitz** | Besitzmatrix, Entfernungscheckliste, Statuswerte. | 1/2 | R7, D8 |
| **Session-State** | Worktree-taugliche `SESSION.md`/`JOURNAL.md`; ein Merge verliert keine lokalen Inhalte und lässt das Worktree bei unsicherer Übergabe stehen. | 1/2 | R8, D5 |

## Etappe 1 – Eigenständigen Skill herstellen und belastbar machen

- Skill-Name, Metadaten, Version und Installationspfade auf `lupysta` abstimmen; Frontmatter und Ordnername müssen übereinstimmen.
- Den `stack-contract-v1` und die Vorlage der Projekt-`AGENTS.md` innerhalb des Skills ablegen, unter `lupysta/resources/pyinit/`. `pyinit` liest ausschließlich aus dem Skill-Verzeichnis.
- `lupysta/scripts/luxuspythonstack.sh` als mitgelieferte Implementierung verwenden; Pfadauflösung auch bei Installation über Symlinks sicherstellen.
- Abhängigkeiten von Dateien außerhalb des Skills auflösen. Besonders wichtig: `pyinit` liest den Vertrag für die erzeugte `AGENTS.md` noch aus einer im neuen Repository fehlenden `luxuspythonstack.md` (`lupysta/scripts/luxuspythonstack.sh:162-166`).
- Alte Links auf `deen-lupysta`, `ai-stack`, `base-system` und `../refactoring` prüfen und durch interne Inhalte oder ausdrücklich benannte optionale Referenzen ersetzen.
- Portabilität herstellen oder die Plattformzusage auf Linux einschränken (D2). Symlink-Installation muss auf der zugesagten Plattform funktionieren.
- `pyinit` transaktional machen: Werkzeuge, Contract, Vorlagen und Python-Version vor dem ersten Schreibzugriff prüfen; `.git`-only als zulässiges Ziel behandeln; `--no-check` anbieten; bei Fehlern keinen unbrauchbaren Rest hinterlassen.
- Bestehende pyinit-Tests aus der alten Version übernehmen und an die neue Struktur anpassen. Schnelle und langsame Fälle trennen.
- Shellcheck, portablen Link-Checker und CI einrichten; Fresh-Clone-Test ohne `ignore/`.
- `LICENSE` und ggf. `NOTICE` ergänzen; die Lizenzentscheidung bleibt offen (D3).
- Feature-Grundlagen (Feature-Tabelle, Statuswerte, Besitzanteile) in der erzeugten `AGENTS.md` vorbereiten.
- Installation in zwei Schritten dokumentieren: Skill-Ordner in das Skills-Verzeichnis des jeweiligen Harness verlinken; anschließend einen korrekt quotierten `source`-Befehl mit absolutem Pfad zu `luxuspythonstack.sh` ausgeben. Der Nutzer entscheidet selbst, ob und wo er ihn in seine Shell-Konfiguration einbindet.
- Migration vom alten `luxuspythonstack`-Skill und Uninstall dokumentieren; nicht beide Skills parallel installieren.
- Aktualisierung des zentralen Skills und Herkunftsangabe der generierten Projekte dokumentieren.

**Abnahme:** Ein frischer Checkout genügt zur Installation und Projektinitialisierung. Es werden weder das alte Repository noch lokale Links unter `ignore/` benötigt. Installation über Symlink funktioniert in Bash und Zsh. App, CLI-App und Bibliothek lassen sich erzeugen und bestehen ihr `just check`. Schnelle Tests, Shellcheck, Link-Check und CI sind grün. Ein nicht-leeres Ziel wird verständlich abgelehnt; ein Ziel mit `.git` wird akzeptiert.

## Etappe 2 – Dokumentation, Setup und Feature-Modell

- `SKILL.md` auf Einsatzbereich, Grundregeln, kurzen Überblick über die Entwicklungs- und CI/CD-Features sowie ein Rezeptverzeichnis konzentrieren. Detailanleitungen werden erst bei Bedarf geladen.
- Den vollständigen Contract nicht in jede Projekt-`AGENTS.md` kopieren. Erzeugt werden: Profil, Feature-Tabelle, `lupysta_version`, `contract_version` und lokale Regeln.
- Feature-Tabelle mit Feature, Status, Einstiegspunkt, Rezeptname und Prüfbefehl erzeugen. Statuswerte: `available`, `removed`, `not-generated`, `legacy`.
- `just lupysta-status` bereitstellen: liest ausschließlich die Feature-Tabelle, prüft Tabellenformat, Statuswerte und deklarierte Einstiegspunkte, ändert nichts, Exitcode 0/1. Datei-, Gruppen- und Entfernungsprüfungen bleiben bei den Rezepten und beim Agenten; kein zweiter Katalog.
- Regeln für Widersprüche zwischen Feature-Tabelle und Projektzustand festlegen: tatsächliche Verfügbarkeit prüfen, Abweichungen sichtbar machen und bei Änderungen im Aufgabenrahmen die Tabelle mitpflegen.
- `references/daily-commands.md` auflösen: Befehlsübersicht in den Skill, weiterführende Abläufe und Fehlerdiagnose in die passenden Referenzen übertragen.
- Folgende Referenzen erstellen beziehungsweise überarbeiten:
  - `lupysta-setup.md`
  - `e2e-testing.md`
  - `e2e-agent-testing.md`
  - `rate-and-refactor.md`
- Die relevanten Inhalte der alten Setup-Kapitel prüfen und verallgemeinern.
- Setup in nachvollziehbare Ausbaustufen gliedern: Projektentwicklung, Zsh-Komfort mit Antidote/Powerlevel10k, optional Mamba/Jupyter.
- Voraussetzungen vollständig nennen; keine stillschweigende Abhängigkeit von Concierge, shlib, MCPHub oder vorherigen Installationskapiteln.
- Einrichtungsschritte mit Zustandsprüfung, Änderung und Erfolgskontrolle beschreiben. Vorhandene Konfigurationen erhalten und doppelte Einträge bei Wiederholung vermeiden.
- Rückwärtskompatibilität dokumentieren: ältere Projekte ohne Feature-Tabelle bleiben nutzbar; Rezepte nennen das Mindest-Schema (D10).

**Abnahme:** Mensch und Agent können die Einrichtung anhand der Anleitung nachvollziehen. Die Projektentwicklung benötigt weder Mamba/Jupyter noch einen bestimmten Prompt. Der Skill vermittelt das Gesamtbild; die Projekt-`AGENTS.md` benennt die konkrete Feature-Auswahl. Unbenötigte Rezepte müssen nicht geladen werden. Alle internen Links sind gültig. `just lupysta-status` meldet im erzeugten Projekt keine Abweichung und erkennt eine künstlich erzeugte Lücke.

## Etappe 3 – Designbewertung und Refactoring modernisieren

- Lions aktuellen Stand mit dem bereits in pyinit vorhandenen Mechanismus vergleichen und die allgemein verwendbaren Teile übernehmen.
- Einen schlanken Design-Report bereitstellen (Richtwert 200–300 Zeilen): `run`, `save <name>`, `compare <name>`, Ablage unter `.refactor/`. Ein `incomplete`-Marker kennzeichnet unvollständige Messungen.
- `just rate-design` sowie benannte Baselines und Vergleiche über `save <name>` und `compare <name>` bereitstellen.
- `just refactor <name>` prüft den Ausgangszustand, führt das Quality Gate aus, sichert eine Baseline und erstellt einen Branch in einem separaten Worktree. Der Befehl verändert selbst keinen Anwendungscode.
- Den Ablauf mit `just merge` abstimmen: Merge aus dem Haupt-Checkout, expliziter oder eindeutig ermittelter Branch, verständliche Ablehnung bei Mehrdeutigkeit oder divergiertem `main`.
- Session-Dateien beim Merge schützen (D5): Vor dem Entfernen eines Worktrees prüfen, ob `SESSION.md` oder `JOURNAL.md` lokale Inhalte enthalten. Ohne nachweislich sichere Übergabe das Worktree nicht entfernen, den Pfad nennen und den Git-Merge getrennt von der ausstehenden Bereinigung melden. Vorhandene Dateien im Haupt-Checkout nie überschreiben, kein Force-Remove.
- Eine automatische Übergabe erst später und nur mit eigener Spezifikation für Journal-Zusammenführung, gleiche Einträge, Session-Konflikte, Sicherung und Schreibfehler umsetzen.
- Baselines unter `.refactor/baselines/` vor `just clean` schützen.
- Messwerte bleiben Hinweise; technische Fehler bei der Messung führen zu einem Fehlerstatus.
- Arbeitsregeln und Grenzen der Metriken in `rate-and-refactor.md` beschreiben. Keine Abhängigkeit von einem nicht mitgelieferten Refactoring-Skill.

**Abnahme:** Ein frisch erzeugtes Projekt kann gemessen werden. Baseline und Vergleich funktionieren; Bereinigung erhält Baselines. Refactoring-Vorbereitung lehnt einen ungeeigneten Ausgangszustand verständlich ab. Nach einem Refactor-Lauf kann gemergt werden, ohne lokale Session- oder Journal-Inhalte zu verlieren; ein unsicherer Fall bleibt als Worktree sichtbar und wird verständlich gemeldet.

## Etappe 4 – E2E-Grundgerüst integrieren

- `tests/e2e/` mit README, gemeinsamen Fixtures und einer geeigneten Containerdefinition anlegen.
- Root-`tests/conftest.py` mit env-gesteuertem Collection-Gate (`LUPYSTA_E2E=1`) anlegen.
- `just test-e2e` mit einem mitgelieferten Runner verbinden. Der Runner prüft zuerst auf Testdateien, dann den Docker-Preflight.
- Lions Aufbau auf den allgemeinen Kern reduzieren: Wegwerfcontainer, installiertes Projekt, temporäre Testdaten, Logs/JUnit-Bericht und zuverlässiges Aufräumen.
- Gewählte Python-Version und die verschiedenen pyinit-Projekttypen berücksichtigen.
- Bei null Tests eine eindeutige Meldung und Exitcode 0 liefern, ohne Docker vorauszusetzen. Nur den echten Leerfall tolerieren; Collection- und Ausführungsfehler bleiben Fehler.
- Reale Tests innerhalb der vorgesehenen isolierten Umgebung ausführen; keine Host-Mounts für veränderbare Benutzerdaten.
- Dependency-Group `e2e`, Dockerfile-Anforderungen (unprivilegierter Nutzer, `/artifacts`, `uv sync --locked --no-dev --group e2e`) umsetzen.
- Artefakte in `.gitignore` und `just clean` berücksichtigen.
- Einen optionalen separaten CI-Workflow vorbereiten.

**Abnahme:** Der leere Aufruf funktioniert ohne Docker. Ein ergänzter minimaler Test läuft im Container; ein fehlerhafter Test lässt den Aufruf scheitern. Normale Tests sammeln die Suite nicht ein.

## Etappe 5 – Agent-Testgrundgerüst integrieren (bedingt)

**Vorbedingung:** Go/No-Go nach Etappe 4 (D6). Ohne Go wird die Etappe nicht begonnen; die Roadmap bleibt bis dahin gültig.

- `tests/agent/` und `just test-agent` bereitstellen. Der neue allgemeine Aufbau verwendet `agent` statt Lions bisherigem Ordnernamen `agentic`.
- Einen minimalen Kern (Richtwert ≤600 Zeilen) mit konfigurierbarem Testziel, OpenAI-kompatiblem Provider, Zeit- und Schrittlimits sowie Diagnoseaufzeichnung bereitstellen.
- CLI/HTTP zuerst; MCP ist optional und kein Pflichtbestandteil.
- Einen klaren Erweiterungspunkt für projektspezifische Interaktion und Ergebnisprüfung definieren.
- Zusätzliche Abhängigkeiten getrennt halten (Gruppe `agent`) und ihre Installation dokumentieren. Keine Providerzugriffe beim normalen Testlauf.
- Keine echten Agent-Testdateien standardmäßig erzeugen. Fake-Provider-Tests decken Fehlerbehandlung und Limits ab.
- Null Tests ohne Zugangsdaten oder laufenden Modellserver erfolgreich melden. Bei tatsächlich vorhandenen Agent-Tests führen fehlende Voraussetzungen zu verständlichen Fehlern, nie zu stillem Skip.
- Technische Szenarioprüfungen von modellabhängiger Qualitätsbewertung unterscheiden; Ergebnisse bleiben eine Ergänzung zum deterministischen Quality Gate.
- Kein vollständiger Lion-Port; Lions Szenariokatalog wird nicht übernommen.

**Abnahme:** Der leere Aufruf funktioniert ohne Provider. Der Harness lässt sich ohne Live-Modell auf Fehlerbehandlung und Limits prüfen. Ein dokumentierter Live-Lauf belegt den vorgesehenen Einstieg. Der Report führt Preflight-Fehler als eigenen Status.

## Etappe 6 – Stabilisierung, Feature-Entfernung und Release

- Entfernungstests an repräsentativen Projekten durchführen: vollständig, `--no-e2e`, `--no-agent`, `--no-refactor`, `--minimal` sowie nachträglich entferntes Feature.
- Die Integrationsmatrix für App, CLI-App und Bibliothek, Bash/Zsh sowie Symlink-Installation vervollständigen.
- Doku, Feature-Tabelle, Ignore-Regeln und `just clean` aufeinander abstimmen.
- Die unterstützte Umgebung ausdrücklich benennen (Betriebssystem, Python, `uv`, `just`, `git`, `bash`, `direnv`, Docker, Provider).
- `CHANGELOG` und Release-Checkliste anlegen; die Lizenz vor der Veröffentlichung klären.
- Einen stabilen Stand veröffentlichen.

**Abnahme:** Das Entfernen optionaler Features lässt die verbleibenden Abläufe grün. Doku und tatsächliche Befehle stimmen überein. Standardprüfungen benötigen weder Docker noch einen Modellanbieter. Die Release-Checkliste ist vollständig abgearbeitet.

## Offene Entscheidungen

Status gemäß [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md). Diese Punkte sind nicht beschlossen.

| ID | Thema | Status | Empfehlung | Blockiert |
| --- | --- | --- | --- | --- |
| D2 | Unterstützte Plattformen | Open | Linux als Referenz, `platforms: [linux]`; macOS nur mit CI. | Etappe 1 |
| D3 | Lizenz | Open | Apache-2.0 mit `NOTICE` für MIT-Teile. | Etappe 1/6 |
| D4 | Template-/Scaffolder-Mechanismus | Open | Vorlagen unter `resources/pyinit/` plus kleiner Renderer; Python-Scaffolder nur als Spike. | Etappe 1 |
| D5 | Session-Zustand über Worktrees | Proposed | Root-Dateien behalten; ohne sichere Übergabe Worktree stehen lassen, nichts überschreiben. | Etappe 3 |
| D9 | Design-Report-Umfang | Open | Schlanke Version zuerst (200–300 Zeilen). | Etappe 3 |
| D13 | Feature-Standardauswahl | Open | Alle optionalen Features vorbereiten, mit Opt-out-Flags. | Etappe 1/2 |

## Risiken

| Risiko | Wirkung | Gegenmaßnahme |
| --- | --- | --- |
| **Agent-Harness-Umfang** | Etappe 5 überschreitet Kapazität deutlich (Lion ca. 2.900 Zeilen). | Minimaler Kern, Go/No-Go nach Etappe 4, kein Lion-Port. |
| **Plattformdrift** | Zusage `darwin` bleibt ungetestet; Nutzer scheitern still. | Linux-only bis CI-Abdeckung (D2). |
| **Feature-Drift** | `AGENTS.md`, Rezepte und tatsächliche Dateien laufen auseinander. | Feature-Tabelle, `just lupysta-status`, Entfernungstests. |
| **Ungepinnte Toolchain** | Projekte sind zeitabhängig nicht reproduzierbar. | Versionen dokumentieren/pinnen, CI-Lauf (D11). |
| **Lizenz** | Veröffentlichung ohne klare Rechte. | Entscheidung vor Release (D3), `NOTICE` für MIT-Teile. |
| **Alter und neuer Skill parallel** | Agenten laden widersprüchliche Anleitungen. | Migration/Uninstall dokumentieren, nicht beide installieren (D12). |

## Vorerst außerhalb des Umfangs

- Automatische Aktualisierung bestehender pyinit-Projekte.
- Mehrere parallel im Harness geladene lupysta-Versionen.
- Ein allgemeines Skill- oder Projekt-Migrationsframework.
- Automatische Feature-Erkennung oder ein eigener Rezeptmanager; zunächst genügen die Feature-Tabelle und die Prüfung der angegebenen Einstiegspunkte.
- Vollständiger Lion-Port, Lions Szenariokatalog und Lions Spezialausstattung wie tmux-Liveansicht, E2E-Sharding oder konkrete Systemverwaltungsszenarien als Standardausstattung.
- Änderungen am bestehenden Lion-Projekt oder an der alten Skill-Version in `deen-lupysta`.
