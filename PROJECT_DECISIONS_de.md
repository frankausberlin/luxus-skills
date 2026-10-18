# Projekt-Entscheidungen: lupysta

Stand: 18. Oktober 2026  
Bezug: [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md), [`PROJECT_REVIEW_de.md`](PROJECT_REVIEW_de.md), [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md), [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md)

## Zweck

Dieses Dokument ist ein schlankes Entscheidungslog (ADR-light). Jede Entscheidung nennt Kontext, Optionen, Empfehlung, Status und Konsequenzen. Die Roadmap übernimmt die Status unverändert; offene Entscheidungen dürfen dort nicht als beschlossen dargestellt werden.

## Statuslegende

| Status | Bedeutung |
| --- | --- |
| **Accepted** | In der Roadmap bereits als Richtung festgehalten und bestätigt. |
| **Proposed** | Vorschlag aus dem Review; vom Nutzer zu bestätigen. |
| **Open** | Entscheidung offen; blockiert die jeweilige Etappe. |
| **Rejected** | Verworfen; wird nicht weiterverfolgt. |

## Übersicht

| ID | Thema | Status | Empfehlung |
| --- | --- | --- | --- |
| D1 | Skill-Name und Skriptname | Accepted | Skill/Frontmatter `lupysta`; Skriptdatei `luxuspythonstack.sh` behalten, optional `lupysta.sh` als Alias. |
| D2 | Unterstützte Plattformen | Open | Linux als Referenz; macOS nur mit CI- und Testabdeckung. Bis dahin `platforms: [linux]`. |
| D3 | Lizenz | Open | Apache-2.0 passend zu `deen-lupysta`; Lion-MIT-Teile mit Attribution/`NOTICE`. Alternativ MIT nur nach sauberer Relizenzierung. |
| D4 | Template-/Scaffolder-Mechanismus | Open | Vorlagen unter `resources/pyinit/` plus kleiner Renderer; Python-Scaffolder als optionaler Spike, nicht als Voraussetzung. |
| D5 | Session-Zustand über Worktrees | Proposed | `SESSION.md`/`JOURNAL.md` bleiben im Projekt-Root und gitignored. Der Merge verliert keine lokalen Inhalte: Ohne nachweislich sichere Übergabe bleibt das Worktree stehen; vorhandene Zieldateien werden nie überschrieben. |
| D6 | Agent-Harness-Umfang | Proposed | Minimaler Kern (Richtwert ≤600 Zeilen) statt Lion-Port; Go/No-Go nach Etappe 4. |
| D7 | E2E/Agent-Collection und Null-Tests | Proposed | Root-`tests/conftest.py` mit env-gesteuertem Ignore; keine echten Testdateien standardmäßig; Runner prüfen zuerst auf `test_*.py`; Exitcode 5 nur bei vorhandenen Dateien als Fehler. |
| D8 | Feature-Modell und Statusbefehl | Proposed | Feature-Tabelle in `AGENTS.md`; `just lupysta-status` prüft deklarierte Einstiegspunkte, keine automatische Feature-Erkennung, keine Reparatur. |
| D9 | Design-Report-Umfang | Open | Schlanke Version (Richtwert 200–300 Zeilen) für Messung, Baseline, Vergleich; Vollversion später. |
| D10 | Versionierung und Kompatibilität | Proposed | `lupysta_version` + `contract_version`/Feature-Schema in `AGENTS.md`; keine automatische Aktualisierung; Rezepte nennen Mindest-Schema. |
| D11 | Reproduzierbarkeit und Voraussetzungen | Proposed | Getestete Tool-Versionen dokumentieren/pinnen; Netzwerk, `uv`, `just`, `git`, `bash`, `direnv`/`layout_uv`, Docker und Provider klar nennen; regelmäßiger CI-Lauf. |
| D12 | Installation, Migration, Koexistenz | Proposed | Symlink + `source`; optionaler `pyinit`-Wrapper für nicht-interaktive Shells; alten `luxuspythonstack`-Skill entlinken; nicht beide installieren; Uninstall dokumentieren. |
| D13 | Feature-Standardauswahl | Open | Alle optionalen Features standardmäßig vorbereiten, mit `--no-e2e`, `--no-agent`, `--no-refactor`, `--minimal`; Alternative: nur Core + Refactor standardmäßig. |
| D14 | Test- und CI-Strategie | Proposed | Alte `test-pyinit.sh` in Etappe 1 übernehmen; schnell/langsam trennen; Link-Checker, Shellcheck, Fresh-Clone-Test, Integrationsmatrix; macOS nur bei Plattformzusage. |
| D15 | Sprache | Accepted | Code, Skill und reguläre Dokumentation englisch; Projektmanagement, Roadmap, Review, Entscheidungen, Features und Testing deutsch. |

## D1 – Skill-Name und Skriptname

**Status:** Accepted

**Kontext:** Der Ordner heißt `lupysta`, das Frontmatter nennt noch `luxuspythonstack` (`lupysta/SKILL.md:2`). Das Skript `lupysta/scripts/luxuspythonstack.sh` enthält die etablierten Funktionen `pyinit`, `act`, `jupyter-launcher` und `jl`.

**Optionen**

1. Alles auf `lupysta` umbenennen, auch die Skriptdatei.
2. Skill auf `lupysta`, Skriptdatei aus Kompatibilitätsgründen behalten.
3. Beide Namen dauerhaft parallel führen.

**Empfehlung:** Option 2. Frontmatter und Ordnername `lupysta`; Skriptdatei `luxuspythonstack.sh` behalten, optional `lupysta.sh` als Alias oder Wrapper.

**Konsequenzen:** Bestehende `source`-Zeilen und Shell-Konfigurationen funktionieren weiter. Die Dokumentation nennt genau einen kanonischen Pfad. Ein Alias muss klar als solcher gekennzeichnet sein, damit keine zweite Implementierung entsteht.

## D2 – Unterstützte Plattformen

**Status:** Open

**Kontext:** `lupysta/SKILL.md:12` verspricht Linux und macOS. Das Skript nutzt GNU-spezifische Werkzeuge (`readlink -f`, `sed -i`, `mapfile`, `realpath -m`, siehe [`PROJECT_REVIEW_de.md`](PROJECT_REVIEW_de.md), R2). Ein macOS-Job existiert nicht.

**Optionen**

1. Linux als Referenzplattform, `platforms: [linux]`; macOS erst nach CI-Abdeckung.
2. Portabilität sofort herstellen und macOS mit CI zusagen.
3. macOS-Zusage entfernen und dauerhaft Linux-only bleiben.

**Empfehlung:** Option 1. Portabilität schrittweise herstellen, macOS erst mit CI- und Testabdeckung wieder zusagen.

**Konsequenzen:** Die Zusage ist ehrlich und prüfbar. Nutzer auf macOS erhalten einen klaren Hinweis statt stiller Fehlschläge. Option 2 kostet zusätzlichen Aufwand in Etappe 1.

## D3 – Lizenz

**Status:** Open

**Kontext:** Das alte Repository steht unter Apache-2.0, Lion unter MIT. Das neue Repository hat keine `LICENSE`; `lupysta/SKILL.md:11` nennt MIT. Übernommene Teile stammen aus beiden Quellen.

**Optionen**

1. Apache-2.0 mit `NOTICE` für MIT-Teile.
2. MIT für das Gesamtprojekt nach sauberer Relizenzierung.
3. Getrennte Lizenzen je Unterverzeichnis.

**Empfehlung:** Option 1. Apache-2.0 passt zur Herkunft des größten Teils; MIT-Teile werden in `NOTICE` attribuiert.

**Konsequenzen:** Klare Nutzungsrechte vor der ersten Veröffentlichung. Option 2 verlangt die Prüfung aller übernommenen Dateien und ist nur mit Zustimmung der Rechteinhaber zulässig. Option 3 erhöht den Dokumentationsaufwand im Alltag.

## D4 – Template-/Scaffolder-Mechanismus

**Status:** Open

**Kontext:** `pyinit` erzeugt Dateien heute über Heredocs im Shell-Skript. Der Contract lag außerhalb des Repositories. Etappe 1/2 braucht eine Ablage für `AGENTS.md`, Feature-Tabellen und Test-READMEs.

**Optionen**

1. Vorlagen unter `lupysta/resources/pyinit/` plus kleiner Renderer im Shell-Skript.
2. Vollständiger Python-Scaffolder (z. B. Cookiecutter-ähnlich) als Ersatz.
3. Alles weiterhin als Heredocs im Shell-Skript.

**Empfehlung:** Option 1. Vorlagen als Dateien mit klar definierten Platzhaltern; Rendering bleibt im mitgelieferten Skript. Ein Python-Scaffolder ist ein optionaler Spike, keine Voraussetzung.

**Konsequenzen:** Vorlagen sind lesbar, testbar und per Link prüfbar. Der Renderer muss Platzhalter, Quoting und Fehlerfälle beherrschen. Option 2 erhöht die Einstiegshürde und verlagert Logik aus dem bestehenden Skript.

## D5 – Session-Zustand über Worktrees

**Status:** Proposed

**Kontext:** `SESSION.md` und `JOURNAL.md` sind gitignored (`lupysta/scripts/luxuspythonstack.sh:448-449`) und liegen pro Worktree. `just refactor` arbeitet im aktuellen Checkout, `just merge` entfernt ein Worktree samt Branch (`:679-681`). Nicht versionierte Dateien gehen dabei verloren. Ein pauschales Kopieren in den Haupt-Checkout kann dort vorhandene Session- oder Journal-Inhalte überschreiben.

**Optionen**

1. Dateien bleiben im Projekt-Root und gitignored. Der Merge entfernt ein Worktree nur, wenn dabei keine lokalen Session- oder Journal-Inhalte verloren gehen; sonst bleibt das Worktree mit Pfad und Hinweis stehen.
2. Dateien bleiben im Projekt-Root; der Merge kopiert sie automatisch und definiert dafür Zusammenführung, Konflikte, Sicherung und Schreibfehler.
3. Dateien versionieren.
4. Session-Zustand außerhalb des Repositories ablegen.

**Empfehlung:** Option 1 für den ersten Stand. Vor dem Entfernen prüfen, ob zu erhaltende lokale Session-Dateien existieren. Solange keine nachweislich sichere Übergabe stattgefunden hat, das Worktree nicht entfernen, seinen Pfad nennen und den erfolgreichen Git-Merge getrennt von der ausstehenden lokalen Bereinigung melden. Vorhandene Dateien im Haupt-Checkout nie überschreiben und kein Force-Remove verwenden. Option 2 ist eine spätere Erweiterung und braucht eine eigene Spezifikation für Journal-Zusammenführung, gleiche Einträge, Session-Konflikte, Sicherung und Schreibfehler.

**Konsequenzen:** Ein Merge verliert keine lokalen Inhalte, ohne eine unausgereifte Zusammenführung zu versprechen. Im Konfliktfall bleibt ein Worktree sichtbar stehen; das ist beabsichtigt und wird verständlich gemeldet. Option 3 erzeugt Rauschen im Verlauf; Option 4 bricht die einfache Projektlokalität.

## D6 – Agent-Harness-Umfang

**Status:** Proposed

**Kontext:** Lions Harness umfasst ca. 2.437 Zeilen (`agent.py`), 362 (`suite.py`) und 136 (`conftest.py`), dazu Ledger und szenariospezifische Fixtures. Die Roadmap beschreibt nur einen „kleinen Kern“.

**Optionen**

1. Minimaler Kern (Richtwert ≤600 Zeilen) mit CLI/HTTP, Limits, Diagnose und Erweiterungspunkt.
2. Portierung des Lion-Harness mit allgemeinem Kern und Lion-Szenarien.
3. Kein Agent-Feature; E2E genügt.

**Empfehlung:** Option 1, mit Go/No-Go nach Etappe 4. MCP bleibt optional.

**Konsequenzen:** Der Aufwand bleibt beherrschbar, das Feature bleibt nützlich. Option 2 bindet erhebliche Kapazität. Option 3 lässt eine bereits erprobte Testform ungenutzt.

## D7 – E2E/Agent-Collection und Null-Tests

**Status:** Proposed

**Kontext:** Marker plus `addopts` filtern erst nach der Collection. Optionale Importe auf Modulebene und fehlende Gruppen brechen den Standardlauf; ohne Testdateien entsteht Exitcode 5.

**Optionen**

1. Root-`tests/conftest.py` mit env-gesteuertem Ignore; keine echten Testdateien standardmäßig; Runner prüfen zuerst auf Dateien.
2. Nur Marker und `addopts`.
3. Eigene pytest-Konfiguration je Suite.

**Empfehlung:** Option 1. Ignore über `LUPYSTA_E2E=1` und `LUPYSTA_AGENT=1`; Exitcode 5 nur im echten Leerfall tolerieren, bei vorhandenen Dateien als Fehler behandeln.

**Konsequenzen:** `just test` und `just check` bleiben unabhängig von Docker und Provider. Die gemeinsame `conftest.py` gehört beiden Suiten und darf erst entfernt werden, wenn keine der beiden bleibt (siehe [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md)).

## D8 – Feature-Modell und Statusbefehl

**Status:** Proposed

**Kontext:** Die Roadmap fordert eine Feature-Tabelle in `AGENTS.md`. Ein Rezeptmanager oder eine automatische Feature-Erkennung ist ausdrücklich außerhalb des Umfangs.

**Optionen**

1. Tabelle plus `just lupysta-status` als reine Prüfung.
2. Automatische Erkennung über Dateisystem und Justfile.
3. Keine Tabelle, nur Prosa in `AGENTS.md`.

**Empfehlung:** Option 1. Statuswerte `available`, `removed`, `not-generated`, `legacy`; der Befehl prüft Tabellenformat, Statuswerte und deklarierte Einstiegspunkte und meldet Abweichungen, ändert aber nichts (Exitcode 0/1). Einzige Datenquelle ist die Feature-Tabelle; Datei-, Gruppen- und Entfernungsprüfungen bleiben bei den Rezepten und beim Agenten. Damit entsteht kein zweiter Katalog. Eine spätere Erweiterung um Datei- und Gruppenprüfungen braucht eine eigene Festlegung von Datenquelle und Pflegeaufwand.

**Konsequenzen:** Agenten erkennen fehlende Features, ohne sie ungefragt wiederherzustellen. Die Tabelle bleibt kurz und pflegbar. Option 2 ist fehleranfällig und widerspricht dem Scope.

## D9 – Design-Report-Umfang

**Status:** Open

**Kontext:** Lions `design_report.py` hat ca. 1.524 Zeilen. Der vorhandene `rate-design`-Block schreibt nur Radon-/Vulture-Berichte, ohne `save`/`compare`.

**Optionen**

1. Schlanker Report (Richtwert 200–300 Zeilen) mit `run`, `save <name>`, `compare <name>`, Baseline-Ablage.
2. Vollständiger Port von Lion.
3. Nur Radon/Vulture-Ausgabe ohne Baselines.

**Empfehlung:** Option 1; die Vollversion erst bei nachgewiesenem Bedarf.

**Konsequenzen:** Etappe 3 bleibt klein und prüfbar. Option 3 erfüllt die Abnahme nicht, Option 2 überzieht sie deutlich.

## D10 – Versionierung und Kompatibilität

**Status:** Proposed

**Kontext:** Die erzeugte `AGENTS.md` nennt nur `stack-contract-v1`, aber keine lupysta-Version. Eine automatische Aktualisierung bestehender Projekte ist ausgeschlossen.

**Optionen**

1. `lupysta_version` und `contract_version`/Feature-Schema in `AGENTS.md`; Rezepte nennen Mindest-Schema.
2. Keine Versionsangabe; Agenten prüfen nur den Projektzustand.
3. Automatische Synchronisierung über einen Updater.

**Empfehlung:** Option 1, ohne automatische Aktualisierung.

**Konsequenzen:** Herkunft und Kompatibilität sind nachvollziehbar; ältere Projekte bleiben gültig. Option 3 widerspricht der Leitentscheidung „Initialisierung statt Aktualisierung“.

## D11 – Reproduzierbarkeit und Voraussetzungen

**Status:** Proposed

**Kontext:** `uv add` löst ohne Versionsangaben auf (`lupysta/scripts/luxuspythonstack.sh:234`). Netzwerk, `uv`, `just`, `git`, `bash`, `direnv`/`layout_uv`, Docker und Provider werden nicht systematisch genannt.

**Optionen**

1. Getestete Versionen dokumentieren und in CI pinnen; Voraussetzungen je Etappe nennen.
2. Ungepinnt lassen und auf aktuelle Toolchains vertrauen.
3. Vollständige Lockfiles für alle Vorlagen.

**Empfehlung:** Option 1. Netzwerkbedarf ausweisen, Docker und Provider nur für die optionalen Suiten verlangen.

**Konsequenzen:** Fehlerbilder werden reproduzierbar. Option 2 erzeugt zeitabhängige Projekte; Option 3 erschwert Aktualisierungen unnötig.

## D12 – Installation, Migration, Koexistenz

**Status:** Proposed

**Kontext:** Installation erfolgt über Symlink plus `source`. Der alte Skill `luxuspythonstack` kann parallel installiert bleiben.

**Optionen**

1. Symlink + `source`, optionaler `pyinit`-Wrapper für nicht-interaktive Shells, dokumentierte Migration und Uninstall.
2. Installer-Skript, das alles automatisch einrichtet.
3. Keine Migrationshinweise.

**Empfehlung:** Option 1. Ausdrücklich: alten Skill entlinken, nicht beide installieren, Uninstall beschreiben. Installationsprüfung über `command -v pyinit` und den Skill-Pfad.

**Konsequenzen:** Nutzer behalten die Kontrolle über ihre Shell-Konfiguration. Option 2 erhöht den Wartungsaufwand und die Fehlerfläche; Option 3 führt zu widersprüchlichen Skill-Versionen (R11).

## D13 – Feature-Standardauswahl

**Status:** Open

**Kontext:** E2E, Agent und Design/Refactor sollen vorbereitet, aber optional sein. Unklar ist, was `pyinit` ohne Flags erzeugt.

**Optionen**

1. Alle optionalen Features vorbereiten, mit `--no-e2e`, `--no-agent`, `--no-refactor`, `--minimal`.
2. Nur Core + Design/Refactor standardmäßig; E2E und Agent nur per Flag.
3. Core als einziger Standard; alles andere nur per Flag.

**Empfehlung:** Option 1, weil die Zusatz-Suiten mit null Tests erfolgreich starten und die Mechanik damit sofort verfügbar ist.

**Konsequenzen:** Projekte erhalten mehr Dateien, dafür funktionierende Einstiegspunkte. Opt-out-Flags müssen getestet und dokumentiert sein. Option 2/3 erzeugen weniger Grundgerüst, verlangen aber spätere manuelle Einrichtung.

## D14 – Test- und CI-Strategie

**Status:** Proposed

**Kontext:** Die alte `test-pyinit.sh` (337 Zeilen) ist eine belastbare Grundlage, fehlt aber im neuen Repository. Es gibt keine CI, keinen Link-Checker, kein Shellcheck.

**Optionen**

1. Alte Suite übernehmen, schnell/langsam trennen, CI mit Link-Check, Shellcheck, Fresh-Clone-Test und Integrationsmatrix.
2. Neue, kleinere Testsuite von Grund auf.
3. Nur manuelle Prüfung vor Releases.

**Empfehlung:** Option 1.

**Konsequenzen:** Regressionen werden früh erkannt; die Suite wächst mit den Etappen. Details in [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md).

## D15 – Sprache

**Status:** Accepted

**Kontext:** Der Skill und die Referenzen sind englisch; Projektnotiz und Roadmap sind deutsch.

**Optionen**

1. Code, Skill, reguläre Dokumentation englisch; Projektmanagement deutsch.
2. Alles englisch.
3. Alles deutsch.

**Empfehlung:** Option 1.

**Konsequenzen:** Die Regel bleibt in allen Dokumenten gleich. Dateinamen folgen dem Muster `PROJECT_*_de.md` für deutsche Projektmanagement-Dokumente.
