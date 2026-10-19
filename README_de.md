> **Deutsche Arbeitsfassung.** Übersetzung von [README.md](README.md). Sie ist ab jetzt die gemeinsame Arbeitsgrundlage; die englische Fassung wird erst nach Abschluss der deutschen Fassung nachgezogen.


# Luxus Skills

Wiederverwendbare Agent-Skills für einen komfortablen, konsistenten Entwicklungsworkflow.

Das *luxoriös* steht für die verwendung komfortabler mechaniken und tools im entwicklungsprozess. Die idee ist, wenn ich einem agenten (=skill+harness) eine umfangreiche und komfortable projektstruktur vorgebe (init) und ihn anweise, diese effektiv einzusetzen (skill), führt dies zur verbesserung der qualität der agentischen arbeit. Vor allem durch automatismen in der ci/cd wird ein hohes maß an stabilität erreicht. 

Ein Luxus Stack (z.b. Python) befindet sich im gleichnamigen folder und besteht aus drei komponenten:
|Komponente|Beschreibung|
|-|-|
|[Stack Dokumentation](Luxus-Python-Stack/Luxus-Python-Stack.md)| Beschreibung des Stacks und seiner einrichtung<br>• Setupanleitung für alle sprachrelevanten tools und umgebungen<br>• Einrichtung der Systemumgebung, ZSH (mit p10k) und integration der stack features<br>• Beschreibung zusätzlicher Stackfeatures, die über den skill hinaus gehen.|
|[Stack Skill](Luxus-Python-Stack/lupysta/SKILL_de.md)| Beschreibung agentischen arbeitens<br>• verwendung der CI/CD<br>• abgleich der dokumentation<br>• Test und refactoring|
|[Stack Script](Luxus-Python-Stack/luxuspythonstack.sh)| Die init-funktion und weiteren stack-spezifischen funktionen<br>• init-funktion zur erzeugung eines vorkonfigurierten projektgerüstes<br>• zusätzliche stack-features außerhalb agentischer zuständigkeit<br>• vorkonfigurierte aliase zum aufruf der script-funktionen|


(noch nicht abgenommener bereich, frank)


Der erste Skill ist **lupysta**, der Luxurious Python Stack: Python-Projektgerüste mit `pyinit`, Entwicklungswerkzeuge und Anleitung für Menschen und Coding-Agenten. 
Er wird aus [deen-lupysta](https://github.com/frankausberlin/deen-lupysta) herausgelöst; Test- und Design-Review-Muster sind von [Lion](https://github.com/frankausberlin/lion) übernommen.



## Aktueller Stand

**In Entwicklung.** Das Repository enthält den ersten Skill, Referenzdokumente und das aus dem vorherigen Stack kopierte Shell-Skript. Die erste Überprüfung ergab, dass die eigenständige Projektinitialisierung noch nicht bereit ist: `pyinit` erwartet weiterhin ein Vertragsdokument, das noch nicht migriert wurde, mehrere Pfade verweisen auf das alte Layout, und die Plattformaussage ist nicht durch die Implementierung gedeckt.

Die [Überprüfung](PROJECT_REVIEW_de.md) dokumentiert die Befunde und das empfohlene schlanke MVP. Stufe 1 der [Roadmap](PROJECT_ROADMAP_de.md) — den Skill eigenständig und zuverlässig zu machen — ist der nächste Schritt. Die [Projektnotizen](PROJECTNOTE_de.md) bewahren die ursprünglichen Ideen.

## Projektdokumente

| Dokument | Inhalt | Sprache |
| --- | --- | --- |
| [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md) | Stufen, Abnahmekriterien, Risiken, offene Entscheidungen | Deutsch |
| [`PROJECT_REVIEW_de.md`](PROJECT_REVIEW_de.md) | Befunde R1–R15 mit Nachweisen und Empfehlungen | Deutsch |
| [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md) | Entscheidungsprotokoll D1–D15 mit Status, Optionen, Konsequenzen | Deutsch |
| [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md) | Feature-Modell, Dateieigentum, Entfernung, `lupysta-status` | Deutsch |
| [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md) | Test- und CI-Strategie, Abnahmematrix | Deutsch |
| [`PROJECTNOTE_de.md`](PROJECTNOTE_de.md) | Ursprüngliche Projektnotizen | Deutsch |

Projektmanagement-Dokumente sind auf Deutsch verfasst. Code, der Skill und reguläre Dokumentation wie die README und die Skill-Referenzen werden auf Englisch verfasst; die deutsche Arbeitsfassung liegt jeweils als `*_de.md` daneben.

## Wie lupysta in ein Projekt passt

lupysta wird einmal in der Agent-Harness installiert. Der Skill stellt den vollständigen Entwicklungs- und Auslieferungsworkflow über eine kurze Übersicht und detaillierte Rezepte bereit, die bei Bedarf geladen werden.

Die `AGENTS.md` jedes Projekts hält dessen lokale Regeln und die in diesem Projekt verfügbaren Features fest, anhand einer Feature-Tabelle mit Status, Einstiegspunkt, Rezeptname und Verifikationsbefehl. Ein geplantes Kommando `just lupysta-status` prüft, ob die deklarierten Einstiegspunkte existieren; es meldet Abweichungen und repariert niemals etwas. Der Agent nutzt diese Informationen, um die relevanten Rezepte im installierten Skill zu finden, und prüft dann die tatsächlichen Projektbefehle, bevor er sie verwendet. Die generierte `AGENTS.md` enthält außerdem `lupysta_version` und `contract_version`.

| Ort | Verantwortlichkeit |
| --- | --- |
| `lupysta/SKILL.md` | Workflow-Überblick, gemeinsame Regeln und Rezeptindex |
| Skill-Referenzen | Detaillierte Einrichtungs- und Feature-Rezepte, bei Bedarf geladen |
| Projekt-`AGENTS.md` | Projektkontext, lokale Regeln, Feature-Tabelle mit Status und Einstiegspunkten |
| Projekt-`Justfile` und Skripte | Ausführbare Implementierung der ausgewählten Features |

Es gibt keinen zweiten lupysta-Skill innerhalb jedes generierten Projekts. Der installierte Skill leitet eine effiziente Agentenarbeit an; generierte Befehle verwenden projektlokale Dateien.

Projekte können Features entfernen, die sie nicht benötigen. Ihre Feature-Tabelle, Konfiguration, Abhängigkeiten und Dokumentation müssen dann aktualisiert werden, während der verbleibende Workflow weiter funktioniert. Entfernte Features bleiben mit Status `removed` sichtbar, damit Agenten sie nicht automatisch neu erstellen. Die Regeln zur Dateiverantwortung und die Checkliste zum Entfernen stehen in [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md).

## Geplante Projekt-Features

Das Zielgerüst umfasst:

- **Kern:** Python-Anwendungen, Typer-CLI-Anwendungen und typisierte Bibliotheken, verwaltet mit uv, mit `just check` als Einstiegspunkt.
- **Design und Refactoring:** ein schlanker Design-Bericht mit gespeicherten Baselines und Vergleichen sowie Refactoring-Vorbereitung in einem separaten Worktree.
- **E2E-Tests:** eine optionale Suite in Wegwerf-Containern unter `tests/e2e/`, gestartet mit `just test-e2e`.
- **Agenten-Tests:** eine optionale Suite unter `tests/agent/`, gestartet mit `just test-agent`, mit begrenzter Ausführung, Provider-Preflight und diagnostischen Artefakten.
- **Dokumentation:** `docs-lib` für Bibliotheken und `docs-cli` für CLI-Anwendungen, mit `just docs` und `just docs-check`.
- **Umgebungseinrichtung:** dokumentierte, optionale `setup-zsh`- und `setup-mamba`-Anleitung für die Python-Toolchain, Zsh-Komfortfunktionen und Mamba/Jupyter-Umgebungen.

Beide optionalen Testsuiten starten erfolgreich mit null Tests, ohne Docker oder einen Modell-Provider zu benötigen. Sie werden durch ein Gate in einer `tests/conftest.py` im Wurzelverzeichnis aus der pytest-Sammlung herausgehalten, nicht allein durch Marker, und bleiben außerhalb der gewöhnlichen Tests, des Standard-Qualitätsgates und der Standard-CI. Das Hinzufügen echter Tests aktiviert ihre dokumentierten Voraussetzungen; fehlende Voraussetzungen schlagen dann sichtbar fehl, statt übersprungen zu werden.

Dies sind die beabsichtigten Fähigkeiten, keine Behauptung, dass die Migration abgeschlossen ist.

## Installationsmodell

Das Folgende beschreibt den angestrebten Installationsablauf. Schließe ihn ab, sobald die eigenständige Migration bereit ist.

1. Dieses Repository an einen stabilen Ort klonen und das Verzeichnis `lupysta/` in das von der Harness verwendete Skill-Verzeichnis verlinken. Einen von dieser Harness unterstützten Namen verwenden; `lupysta` ist der beabsichtigte Skill-Name.
2. Den generierten `source`-Befehl mit dem absoluten Pfad zu `lupysta/scripts/luxuspythonstack.sh` ermitteln. In der Shell ausführen oder in die bevorzugte Shell-Konfiguration aufnehmen.

Zum Beispiel, mit an den eigenen Checkout und die Harness angepassten Pfaden:

```sh
ln -s "/absolute/path/to/luxus-skills/lupysta" "/absolute/path/to/harness/skills/lupysta"
source "/absolute/path/to/luxus-skills/lupysta/scripts/luxuspythonstack.sh"
```

Du entscheidest, wohin der Shell-Befehl gehört. Ein bestimmtes shlib-Layout ist nicht erforderlich. Das Laden des Agenten-Skills und das Einbinden der Shell-Funktionen sind getrennte Schritte.

Für nicht-interaktive Shells ist ein kleiner Wrapper geplant, der vor dem Aufruf von `pyinit` `luxuspythonstack.sh` einbindet; er ist optional und ersetzt den `source`-Schritt für die interaktive Nutzung nicht.

Die Skriptdatei behält aus Kompatibilitätsgründen den Namen `luxuspythonstack.sh`; der Skill-Name ist `lupysta`.

### Migration vom alten Skill

- Den alten `luxuspythonstack`-Skill und lupysta nicht gleichzeitig installieren. Den alten Skill-Link aus jeder Harness entfernen, bevor `lupysta` verlinkt wird.
- Bestehende Projekte, die mit dem alten `pyinit` erstellt wurden, werden nicht automatisch aktualisiert. Sie unverändert lassen und ihre vorhandenen Projektbefehle verwenden.
- Zum Deinstallieren den Skill-Link aus der Harness entfernen und die `source`-Zeile aus der Shell-Konfiguration löschen. Generierte Projekte sind unabhängig vom installierten Skill.

### Unterstützte Umgebung

Linux ist die Referenzplattform. macOS ist derzeit keine unterstützte Zusage: Im Skript werden weiterhin mehrere GNU-spezifische Werkzeuge verwendet. macOS-Unterstützung kommt erst mit CI- und Testabdeckung zurück (siehe Entscheidung D2 in [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md)).

Der zentrale Skill wird in diesem Checkout gepflegt. Eine Aktualisierung überschreibt keine bestehenden Projekte. `pyinit` initialisiert neue Projekte; projektspezifische Anpassungen bleiben bei den jeweiligen Projekten.

## Repository-Aufbau

| Pfad | Inhalt |
| --- | --- |
| [`lupysta/SKILL.md`](lupysta/SKILL.md) | Aktueller Einstiegspunkt des Skills; deutsche Arbeitsfassung: [`SKILL_de.md`](lupysta/SKILL_de.md) |
| [`lupysta/references/`](lupysta/references/) | Detaillierte Anleitung, die zu Rezepten umorganisiert wird |
| [`lupysta/scripts/`](lupysta/scripts/) | Mitgelieferte Shell-Funktionen, einschließlich pyinit |
| `lupysta/resources/pyinit/` | Geplante Gerüstvorlagen, einschließlich `AGENTS.md`-Vertrag und Feature-Tabelle |
| `tests/` | Geplante Testsuite für das Skill-Repository |
| `.github/workflows/` | Geplante CI: Tests, shellcheck, Link-Prüfung, Fresh-Clone-Test |
| [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md) | Umsetzungsplan und Abnahmekriterien |
| [`PROJECT_REVIEW_de.md`](PROJECT_REVIEW_de.md) | Überprüfungsbefunde und Empfehlungen |
| [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md) | Entscheidungsprotokoll |
| [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md) | Feature-Modell und Entfernungsregeln |
| [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md) | Test- und CI-Strategie |
| [`PROJECTNOTE_de.md`](PROJECTNOTE_de.md) | Ursprüngliche Projektnotizen |

Der optionale lokale Ordner `ignore/` ist von Git ausgeschlossen und für Installation, Nutzung oder Test des Skills nicht erforderlich. Kein Dokument und kein Skript darf davon abhängen.

## Entwicklung

Die Skill-Übersicht knapp halten und detaillierte Verfahren in Referenzen auslagern. Generierte Features modular halten, ihre Voraussetzungen dokumentieren und prüfen, dass das Entfernen eines optionalen Features den verbleibenden Workflow nutzbar lässt.

Vor dem Zusammenführen einer Änderung:

- Die schnelle Testsuite ausführen (`bash tests/test-pyinit.sh --fast`, sobald sie migriert ist).
- shellcheck auf den Shell-Dateien und den Link-Prüfer auf allen Markdown-Dateien ausführen.
- Den Fresh-Clone-Test prüfen: Ein Checkout ohne `ignore/` muss den Skill installieren und ein Projekt initialisieren können.
- Prüfen, dass die Feature-Tabelle in `AGENTS.md` jedes generierten Projekts mit den tatsächlich erstellten Dateien und Rezepten übereinstimmt.

Die vollständige Strategie einschließlich E2E- und Agenten-Suiten steht in [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md).

Eine `LICENSE`-Datei und, falls nötig, ein `NOTICE` für Teile von Drittanbietern fehlen noch. Die Lizenzwahl ist offen; die Überprüfung empfiehlt Apache-2.0 mit Namensnennung für MIT-Teile. Siehe Entscheidung D3 in [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md).

Code, Skills und reguläre Dokumentation werden auf **Englisch** verfasst. Projektnotizen, die Roadmap, die Überprüfung, Entscheidungen, Features, Testing und Agenten-Gespräche dürfen auf Deutsch sein. Die deutsche Arbeitsfassung der regulären Dokumentation liegt in den `*_de.md`-Dateien.
