# Projekt-Roadmap: lupysta

Stand: 18. Oktober 2026  
Status: abgestimmte Richtung; technische Details werden in der jeweiligen Etappe konkretisiert.

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
- **Nachvollziehbare Herkunft:** Die erzeugte `AGENTS.md` nennt die verwendete lupysta-Version; daraus entsteht keine Versionsbindung.
- **Optionale Nutzung:** E2E und Agent werden grundsätzlich vorbereitet, bleiben aber außerhalb von `just test`, `just check` und der standardmäßig erzeugten CI.
- **Sprache:** Skill, Code und reguläre Dokumentation sind englisch. Projektnotiz, diese Roadmap und die Kommunikation dürfen deutsch sein.
- **Bewusste Auslagerung:** Der bisherige Skill in `deen-lupysta` bleibt als alte Version bestehen. Die neue Entwicklung erfolgt in `luxus-skills`.

## Etappe 1 – Eigenständigen Skill herstellen

- Skill-Name, Metadaten, Version und Installationspfade auf `lupysta` abstimmen.
- `lupysta/scripts/luxuspythonstack.sh` als mitgelieferte Implementierung verwenden; Pfadauflösung auch bei Installation über Symlinks sicherstellen.
- Abhängigkeiten von Dateien außerhalb des Skills auflösen. Besonders wichtig: pyinit liest den Vertrag für die erzeugte `AGENTS.md` noch aus einer im neuen Repository fehlenden `luxuspythonstack.md`.
- Die Vorlage für die Projekt-`AGENTS.md` innerhalb des Skills ablegen, beispielsweise unter `resources/pyinit/`. Sie enthält lokale Regeln, Herkunft und Feature-Auswahl; allgemeine Workflow-Anleitungen bleiben im zentralen Skill.
- Alte Links auf `deen-lupysta` prüfen und durch interne Inhalte oder ausdrücklich benannte optionale Referenzen ersetzen.
- Bestehende pyinit-Tests aus `deen-lupysta` übernehmen und an die neue Struktur anpassen.
- Installation in zwei Schritten dokumentieren: Skill-Ordner in das Skills-Verzeichnis des jeweiligen Harness verlinken; anschließend einen korrekt quotierten `source`-Befehl mit absolutem Pfad zu `luxuspythonstack.sh` ausgeben. Der Nutzer entscheidet selbst, ob und wo er ihn in seine Shell-Konfiguration einbindet.
- Aktualisierung des zentralen Skills und Herkunftsangabe der generierten Projekte dokumentieren.

**Abnahme:** Ein frischer Checkout genügt zur Installation und Projektinitialisierung. Es werden weder das alte Repository noch lokale Links unter `ignore/` benötigt. App, CLI-App und Bibliothek lassen sich erzeugen und bestehen ihr `just check`.

## Etappe 2 – Dokumentation und Setup konsolidieren

- `SKILL.md` auf Einsatzbereich, Grundregeln, kurzen Überblick über die Entwicklungs- und CI/CD-Features sowie ein Rezeptverzeichnis konzentrieren. Detailanleitungen werden erst bei Bedarf geladen.
- In der Projekt-`AGENTS.md` eine Feature-Tabelle mit Feature, tatsächlichem Einstiegspunkt und Rezeptname erzeugen. Keine Kopie der vollständigen Skill-Anleitung einbetten.
- Regeln für Widersprüche zwischen Feature-Tabelle und Projektzustand festlegen: tatsächliche Verfügbarkeit prüfen, Abweichungen sichtbar machen und bei Änderungen im Aufgabenrahmen die Tabelle mitpflegen.
- `references/daily-commands.md` auflösen: Befehlsübersicht in den Skill, weiterführende Abläufe und Fehlerdiagnose in die passenden Referenzen übertragen.
- Folgende Referenzen erstellen beziehungsweise überarbeiten:
  - `lupysta-setup.md`
  - `e2e-testing.md`
  - `e2e-agent-testing.md`
  - `rate-and-refactor.md`
- Die relevanten Inhalte aus `deen-lupysta/base-system/07-zsh2-antidote-p10k.md` und `deen-lupysta/eco-systems/02-python.md` prüfen und verallgemeinern.
- Setup in nachvollziehbare Ausbaustufen gliedern: Projektentwicklung, Zsh-Komfort mit Antidote/Powerlevel10k, optional Mamba/Jupyter.
- Voraussetzungen vollständig nennen; keine stillschweigende Abhängigkeit von Concierge, shlib oder vorherigen Installationskapiteln.
- Einrichtungsschritte mit Zustandsprüfung, Änderung und Erfolgskontrolle beschreiben. Vorhandene Konfigurationen erhalten und doppelte Einträge bei Wiederholung vermeiden.

**Abnahme:** Mensch und Agent können die Einrichtung anhand der Anleitung nachvollziehen. Die Projektentwicklung benötigt weder Mamba/Jupyter noch einen bestimmten Prompt. Der Skill vermittelt das Gesamtbild; die Projekt-`AGENTS.md` benennt die konkrete Feature-Auswahl. Unbenötigte Rezepte müssen nicht geladen werden. Alle internen Links sind gültig.

## Etappe 3 – Designbewertung und Refactoring modernisieren

- Lions aktuellen Stand mit dem bereits in pyinit vorhandenen Mechanismus vergleichen und die allgemein verwendbaren Teile übernehmen.
- `just rate-design` sowie benannte Baselines und Vergleiche über `save <name>` und `compare <name>` bereitstellen.
- `just refactor <name>` prüft den Ausgangszustand, führt den Quality Gate aus, sichert eine Baseline und erstellt einen Branch. Der Befehl verändert selbst keinen Anwendungscode.
- Baselines unter `.refactor/baselines/` vor `just clean` schützen.
- Messwerte bleiben Hinweise; technische Fehler bei der Messung führen zu einem Fehlerstatus.
- Arbeitsregeln und Grenzen der Metriken in `rate-and-refactor.md` beschreiben. Keine Abhängigkeit von einem nicht mitgelieferten Refactoring-Skill.

**Abnahme:** Ein frisch erzeugtes Projekt kann gemessen werden. Baseline und Vergleich funktionieren; Bereinigung erhält Baselines. Refactoring-Vorbereitung lehnt einen ungeeigneten Ausgangszustand verständlich ab.

## Etappe 4 – E2E-Grundgerüst integrieren

- `tests/e2e/` mit README, gemeinsamen Fixtures und einer geeigneten Containerdefinition anlegen.
- `just test-e2e` mit einem mitgelieferten Runner verbinden.
- Lions Aufbau auf den allgemeinen Kern reduzieren: Wegwerfcontainer, installiertes Projekt, temporäre Testdaten, Logs/JUnit-Bericht und zuverlässiges Aufräumen.
- Gewählte Python-Version und die verschiedenen pyinit-Projekttypen berücksichtigen.
- Die Suite im Standardlauf bereits von der pytest-Collection ausschließen. Marker allein reichen hierfür nicht.
- Bei null Tests eine eindeutige Meldung und Exitcode 0 liefern, ohne Docker vorauszusetzen. Nur den echten Leerfall tolerieren; Collection- und Ausführungsfehler bleiben Fehler.
- Reale Tests innerhalb der vorgesehenen isolierten Umgebung ausführen; keine Host-Mounts für veränderbare Benutzerdaten.
- Artefakte in `.gitignore` und `just clean` berücksichtigen.

**Abnahme:** Der leere Aufruf funktioniert ohne Docker. Ein ergänzter minimaler Test läuft im Container; ein fehlerhafter Test lässt den Aufruf scheitern. Normale Tests sammeln die Suite nicht ein.

## Etappe 5 – Agent-Testgrundgerüst integrieren

- `tests/agent/` und `just test-agent` bereitstellen. Der neue allgemeine Aufbau verwendet `agent` statt Lions bisherigem Ordnernamen `agentic`.
- Lions Harness in allgemeine Mechanik und Lion-spezifische Szenarien zerlegen; keine vollständige projektspezifische Kopie übernehmen.
- Einen kleinen nutzbaren Kern mit konfigurierbarem Testziel, Provider/Modell, Zeit- und Schrittlimits sowie Diagnoseaufzeichnung bereitstellen.
- Einen klaren Erweiterungspunkt für projektspezifische Interaktion und Ergebnisprüfung definieren. Ein MCP-Beispiel kann den Einstieg zeigen; MCP wird keine Pflicht für alle Projekte.
- Zusätzliche Abhängigkeiten getrennt halten und ihre Installation dokumentieren. Keine Providerzugriffe beim normalen Testlauf.
- Null Tests ohne Zugangsdaten oder laufenden Modellserver erfolgreich melden. Bei tatsächlich vorhandenen Agent-Tests führen fehlende Voraussetzungen zu verständlichen Fehlern.
- Technische Szenarioprüfungen von modellabhängiger Qualitätsbewertung unterscheiden; Ergebnisse bleiben eine Ergänzung zum deterministischen Quality Gate.

**Abnahme:** Der leere Aufruf funktioniert ohne Provider. Die README zeigt einen vollständigen Weg zum ersten echten Agent-Test. Der Harness lässt sich ohne Live-Modell auf Fehlerbehandlung und Limits prüfen; ein dokumentierter Live-Lauf belegt den vorgesehenen Einstieg.

## Etappe 6 – Vorlagen, Gesamtprüfung und Stabilisierung

- Allgemeine Testreferenzen beim Initialisieren nach `tests/e2e/README.md` und `tests/agent/README.md` kopieren. Keine zweite manuell gepflegte Fassung derselben Anleitung anlegen.
- Die kopierten Anleitungen dienen Menschen und Agenten als Einstieg direkt bei den Tests. Sie ersetzen nicht den zentralen Skill und dürfen kein zusätzliches gleichnamiges Skill-Paket bilden. Links und gegebenenfalls Platzhalter beim Erzeugen prüfen beziehungsweise anpassen.
- Projektspezifische Szenarien werden später im jeweiligen Projekt ergänzt; Lions Szenariokatalog wird nicht als allgemeine Anleitung ausgegeben.
- Ausführbare Vorlagen unter `resources/pyinit/` bündeln. Generierte Projektbefehle verwenden lokale Skripte; Agenten verwenden für den Workflow den zentral installierten Skill.
- Features mit ihren Rezepten und benötigten Dateien klar abgrenzen. Gemeinsame Infrastruktur darf beim Entfernen eines Features nicht beschädigt werden; Entfernungsschritte einschließlich Feature-Tabelle dokumentieren.
- Integrationstests für App, CLI-App und Bibliothek, Bash/Zsh sowie Symlink-Installation vervollständigen.
- Leere, erfolgreiche und fehlschlagende Zusatzsuiten sowie ihre Collection-Trennung prüfen. Für E2E- und Agent-Läufe gilt kein eigener Coverage-Mindestwert.
- Herkunftsangaben, Feature-Tabelle, Dokumentation, Ignore-Regeln und Bereinigung aufeinander abstimmen.
- An repräsentativen Projekten prüfen, dass nach dem Entfernen optionaler Features die übrigen Befehle funktionieren und Agenten keine Anleitungen für fehlende Features benötigen.
- Einen stabilen Stand veröffentlichen und die unterstützte Umgebung ausdrücklich benennen.

**Abnahme:** Der zentrale Skill und die projektbezogene Feature-Auswahl greifen zusammen, ohne eine zweite Skill-Version im Projekt. Projektbefehle verwenden ihre lokalen Implementierungen. Standardprüfungen benötigen weder Docker noch einen Modellanbieter. Das Entfernen optionaler Features lässt die verbleibenden Abläufe intakt; Anleitungen und tatsächliche Befehle stimmen überein.

## Während der Umsetzung zu konkretisieren

Diese Punkte blockieren die Roadmap nicht:

- Konkrete Schnittstelle und Umfang des generischen Agent-Harness.
- Containerbasis und unterstützte Betriebssysteme; bestehende Plattformangaben erst nach Prüfung übernehmen.
- Endgültige Ablage und Format der `AGENTS.md`-Vorlage sowie der Feature-Tabelle.
- Zuschnitt und stabile Namen der bedarfsgerecht ladbaren Rezepte.
- Versionsnummer des ersten eigenständigen Releases.

## Vorerst außerhalb des Umfangs

- Automatische Aktualisierung bestehender pyinit-Projekte.
- Mehrere parallel im Harness geladene lupysta-Versionen.
- Ein allgemeines Skill- oder Projekt-Migrationsframework.
- Automatische Feature-Erkennung oder ein eigener Rezeptmanager; zunächst genügen die Feature-Tabelle und die Prüfung der angegebenen Einstiegspunkte.
- Lions Spezialausstattung wie tmux-Liveansicht, E2E-Sharding oder konkrete Systemverwaltungsszenarien als Standardausstattung.
- Änderungen am bestehenden Lion-Projekt oder an der alten Skill-Version in `deen-lupysta`.
