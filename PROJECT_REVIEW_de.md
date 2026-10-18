# Projekt-Review: lupysta

Stand: 18. Oktober 2026  
Bezug: [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md), [`PROJECTNOTE_de.md`](PROJECTNOTE_de.md), [`lupysta/SKILL.md`](lupysta/SKILL.md), [`lupysta/scripts/luxuspythonstack.sh`](lupysta/scripts/luxuspythonstack.sh)

## Zweck und Bezug

Dieses Dokument hält die Ergebnisse der ersten technischen Prüfung des Vorhabens fest. Geprüft wurden:

- die Roadmap gegen den tatsächlichen Stand des Repositories,
- der übernommene Skill (`SKILL.md`, `references/`) gegen die neue Ablagestruktur,
- das mitgelieferte `pyinit`-Skript gegen die Zusage „eigenständig installierbar“,
- die geplanten Etappen 3–6 gegen die als Vorlage dienenden Implementierungen.

Das Review ändert keine Skill-, Skript- oder Testdateien. Es bewertet den Stand und liefert die Grundlage für die Dokumente [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md), [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md) und [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md).

### Hinweis zu Evidenzpfaden

Einzelne Befunde belegen sich über Dateien aus den lokalen Referenz-Checkouts (`ignore/deen-lupysta`, `ignore/lion`). Diese Pfade sind **keine** Projektabhängigkeit: `ignore/` ist per `.gitignore` ausgeschlossen, nicht Teil des Repositories und für Installation, Betrieb oder Tests nicht erforderlich. Die Angaben dienen ausschließlich der Nachvollziehbarkeit der Prüfung.

## Kurzfazit

Das Vorhaben ist sinnvoll und die Richtung der Roadmap trägt. Der zentrale Skill, die Feature-Auswahl in der Projekt-`AGENTS.md` und die entfernbaren Zusatz-Suiten sind ein tragfähiges Modell. Die Dokumentation ist jedoch derzeit optimistischer als der Code:

- **Etappe 1 ist nicht abnehmbar.** `pyinit` bricht in jedem frischen Checkout ab, weil der Contract fehlt, aus dem die `AGENTS.md` erzeugt wird.
- **Die Plattformzusage ist nicht gedeckt.** Mehrere Stellen im Skript und im alten Link-Checker setzen GNU-Werkzeuge voraus.
- **Die Initialisierung ist nicht transaktional** und blockiert typische Wiederholungsfälle.
- **Etappe 3–5 sind unterschätzt.** Besonders der Agent-Harness ist in Lion rund 2.900 Zeilen groß und keine kleine Zerlegung.
- **Querschnittsthemen fehlen:** Lizenz, CI, Tests, Link-Checker, reproduzierbare Tool-Versionen, Migration und Uninstall.

**Empfehlung:** Lean-MVP. Etappe 1 und 2 vollständig abschließen, danach Etappe 3 und 4, Etappe 5 nur nach ausdrücklichem Go/No-Go. Kein vollständiger Lion-Port.

## Stärken

- **Ein zentral installierter Skill:** Es gibt keine zweite Skill-Kopie im erzeugten Projekt; damit entfallen konkurrierende Versionen im Harness.
- **Projektzustand statt Skill-Pfade:** Die Feature-Auswahl in der Projekt-`AGENTS.md` nennt Einstiegspunkte und Rezeptnamen, keine absoluten Skill-Pfade.
- **Optionale Zusatz-Suiten:** E2E und Agent bleiben außerhalb von `just test`, `just check` und der Standard-CI.
- **Entfernbarkeit als Entwurfsziel:** Features sollen mitsamt Konfiguration und Abhängigkeiten entfernt werden können, ohne den Rest zu beschädigen.
- **Initialisierung statt Aktualisierung:** `pyinit` erzeugt neue Projekte und überschreibt keine gewachsenen Projektdateien.
- **Abnahmekriterien je Etappe:** Die Roadmap formuliert prüfbare Ergebnisse statt reiner Aufgabenlisten.
- **Sprachregelung:** Code, Skill und reguläre Dokumentation englisch; Projektmanagement deutsch. Das ist konsistent und praktikabel.
- **Tragfähige Basis:** Das übernommene `pyinit` deckt App, CLI-App und Bibliothek inklusive Docs, Versionierung, Pre-commit und Quality Gate ab; die alte Testsuite ist direkt wiederverwendbar.

## Priorisierte Befunde

Priorität: **P0** blockiert Etappe 1 oder die Veröffentlichung, **P1** blockiert eine spätere Etappe oder die Reproduzierbarkeit, **P2** ist eine Korrektur mit geringerem Risiko.

| ID | Priorität | Bereich | Befund | Empfehlung |
| --- | --- | --- | --- | --- |
| R1 | P0 | Etappe 1 | Etappe 1 nicht abnehmbar: fehlender Contract, alte Links, Namensmismatch, `cw`/`rlb` aus nicht mitgeliefertem Skript. | Contract nach `lupysta/resources/pyinit/` verschieben, Metadaten und Links korrigieren, `cw`/`rlb` entfernen oder mitliefern. |
| R2 | P0 | Portabilität | `readlink -f`, `sed -i`, `mapfile` und `realpath -m` decken die Zusage `platforms: [linux, darwin]` nicht. | Portable Fallbacks einbauen oder vorläufig `platforms: [linux]`; macOS nur mit CI. |
| R3 | P0 | Initialisierung | `pyinit` ist nicht transaktional; ein `.git`-only-Ziel wird abgelehnt. | Vorprüfung vollständig vor dem ersten Schreibzugriff, `.git`-only erlauben, `--no-check`, Aufräum- oder Wiederaufsetzpfad. |
| R4 | P0 | Lizenz | Lizenz unklar: altes Repository Apache-2.0, Lion MIT, neues Repository ohne `LICENSE`, Skill nennt MIT. | Apache-2.0 mit `NOTICE` für MIT-Teile; Entscheidung in D3. |
| R5 | P0 | Tests/CI | Alte `test-pyinit.sh` nicht übernommen; keine CI, kein Link-Checker, kein Shellcheck im Repository. | Suite in Etappe 1 übernehmen, schnell/langsam trennen, CI mit Link-Check, Shellcheck und Fresh-Clone-Test. |
| R6 | P1 | `AGENTS.md` | Voller Contract statt Feature-Tabelle; `lupysta_version` und `contract_version` fehlen; `just lupysta-status` fehlt. | Profil + Feature-Tabelle + lokale Regeln; Versionen ausweisen; Statusbefehl ergänzen (D8, D10). |
| R7 | P1 | Feature-Entfernung | Keine Besitzregeln für gemeinsame pytest-, Justfile- und Doku-Infrastruktur. | Besitzmatrix und Entfernungscheckliste in [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md) verbindlich machen. |
| R8 | P1 | Refactor/Worktree | `refactor`/`merge`/Worktree/Baselines/Session-Dateien widersprechen sich. | Refactor im separaten Worktree oder Merge von jedem Checkout aus; Session-Dateien vor dem Entfernen übernehmen; `clean` erhält Baselines. |
| R9 | P1 | E2E/Agent | Marker allein verhindern keine Collection; Exitcode 5; optionale Abhängigkeiten fehlen im Standardlauf. | Collection-Gate über Root-`conftest.py`, Null-Test-Semantik, Preflight vor Docker/Provider (D7). |
| R10 | P1 | Reproduzierbarkeit | Ungepinnte `uv add`-Auflösung, Netzwerkbedarf, implizites `layout_uv`, fehlende Voraussetzungen. | Getestete Versionen dokumentieren und pinnen, Voraussetzungen nennen, regelmäßiger CI-Lauf (D11). |
| R11 | P1 | Koexistenz | Alter und neuer Skill können gleichzeitig installiert sein. | Migration und Uninstall dokumentieren, nicht beide installieren (D12). |
| R12 | P2 | Agent-Harness | Agent-Harness ist unterschätzt; Lion-Harness ist keine kleine Zerlegung. | Minimaler Kern (Richtwert ≤600 Zeilen), Go/No-Go nach Etappe 4 (D6). |
| R13 | P2 | Design-Report | Lions Report hat ca. 1.500 Zeilen, die Stage-3-Abnahme verlangt deutlich weniger. | Schlanker Report mit `run`/`save`/`compare` (D9). |
| R14 | P2 | Etappenfolge | Etappe 6 enthält Voraussetzungen für Etappe 4/5. | Vorlagen, READMEs und Feature-Grundlagen nach Etappe 1/2 ziehen; Etappe 6 bleibt Stabilisierung/Release. |
| R15 | P2 | Dokumentation | Widersprüche: E2E-Referenz, MCPHub, `ai-stack`, `base-system`, `refactoring`, `daily-commands.md`. | Referenzen neu schreiben, externe Inhalte nur optional und klar gekennzeichnet verlinken. |

## Detailbefunde

### R1 – Etappe 1 ist nicht abnehmbar (P0)

**Evidenz**

- `lupysta/scripts/luxuspythonstack.sh:162-166` – `pyinit` verlangt `${_LUXUS_SCRIPT_DIR}/../luxuspythonstack.md` und bricht ab, wenn die Datei fehlt. Im neuen Repository existiert sie nicht.
- `lupysta/SKILL.md:2` – Frontmatter-Name `luxuspythonstack` statt `lupysta`.
- `lupysta/SKILL.md:12` – `platforms: [linux, darwin]`.
- `lupysta/SKILL.md:27,60,175,200-202,224-229` – Verweise auf `../../luxuspythonstack.md`, `../../scripts/`, `../../base-system/`, `../../ai-stack/`, `../refactoring/SKILL.md` und `scripts/deenlupysta.sh`; MCPHub wird als Voraussetzung dargestellt.
- `lupysta/references/daily-commands.md:3` – verweist auf `../../../luxuspythonstack.md` außerhalb des Repositories.

**Auswirkung:** Jeder `pyinit`-Aufruf scheitert, bevor ein Projekt entsteht. Der Skill beschreibt Funktionen (`cw`, `rlb`), die im ausgelieferten Skript nicht existieren. Ein frischer Checkout ist damit weder installierbar noch benutzbar; die Abnahme von Etappe 1 ist nicht erreichbar.

**Empfehlung:** Den `stack-contract-v1`-Block und die übrigen Vorlagen nach `lupysta/resources/pyinit/` übernehmen und `pyinit` ausschließlich aus dem Skill-Verzeichnis lesen lassen. Frontmatter auf `lupysta` setzen, Version pflegen, Plattformangabe an R2 koppeln. Alle Pfade auf die neue Struktur umschreiben. `cw`/`rlb` entweder in ein mitgeliefertes Skript übernehmen oder aus `SKILL.md` entfernen. MCPHub und externe Setup-Kapitel nur als optionale Referenz nennen.

### R2 – Portabilität nicht gedeckt (P0)

**Evidenz**

- `lupysta/scripts/luxuspythonstack.sh:9` – `readlink -f` zur Auflösung des Quellpfads; kritisch, weil die Installation über Symlinks vorgesehen ist.
- `lupysta/scripts/luxuspythonstack.sh:247,250,1129` – `sed -i` ohne Backup-Suffix.
- `lupysta/scripts/luxuspythonstack.sh:633,640` – `mapfile` in den erzeugten Justfile-Rezepten.
- `lupysta/scripts/luxuspythonstack.sh:1325` – `realpath -m`.
- Alte `check-links.sh`: `mapfile` (Zeilen 30, 68), `readlink -f` (Zeile 95).

**Auswirkung:** macOS liefert BSD-`sed` (verlangt ein Suffix bei `-i`), kein `readlink -f`, kein `realpath -m` und standardmäßig Bash 3.2 ohne `mapfile`. Die Zusage `linux, darwin` ist damit falsch; auf macOS brechen sowohl Installation als auch die erzeugten Projekte.

**Empfehlung:** Entweder portable Ersatzlösungen (Schleifen statt `mapfile`, `cd`/`pwd` statt `readlink -f`, Python oder `perl -pi` statt `sed -i`, `cd`/`pwd` statt `realpath -m`) oder bis zu belastbarer Abdeckung `platforms: [linux]` und Linux als Referenzplattform. macOS erst nach CI- und Testabdeckung wieder zusagen (D2, D14).

### R3 – Initialisierung nicht transaktional (P0)

**Evidenz**

- `lupysta/scripts/luxuspythonstack.sh:134-147` – Zielverzeichnis muss leer sein; ein vorhandenes `.git` genügt für die Ablehnung.
- `lupysta/scripts/luxuspythonstack.sh:1145-1182` – `git init`, `uv sync`, `pre-commit install`, Docs und `just check` laufen nacheinander; bei Fehlern bleibt ein partielles Projekt zurück, der Hinweis darauf steht erst am Ende.
- Ein erneuter Lauf ist wegen der Non-empty-Prüfung ausgeschlossen.

**Auswirkung:** Ein Netzwerk- oder Werzeugfehler hinterlässt ein halbes Projekt, das nur manuell zu bereinigen ist. Der naheliegende Fall „frisch geklontes Repository initialisieren“ (`pyinit .`) scheitert grundsätzlich. Fehlerdiagnose und Wiederholung sind unnötig teuer.

**Empfehlung:** Alle Vorprüfungen (Werkzeuge, Contract, Vorlagen, Python-Version) vor dem ersten Schreibzugriff. Entweder vollständig in ein temporäres Verzeichnis erzeugen und atomar verschieben oder einen abgesicherten Wiederaufsetzpfad anbieten. `.git`-only als zulässigen Zustand behandeln (Inhalt bleibt unangetastet). `--no-check` für Umgebungen ohne Netz oder ohne vollständige Toolchain ergänzen.

### R4 – Lizenz unklar (P0)

**Evidenz**

- Das neue Repository enthält keine `LICENSE`.
- `lupysta/SKILL.md:11` – `license: MIT`.
- Altes Repository: Apache-2.0; Lion: MIT.

**Auswirkung:** Ohne Lizenz ist der Stand rechtlich nicht nutzbar; die Angabe im Skill widerspricht der Herkunft des übernommenen Codes. Eine spätere Relizenzierung ist aufwendiger als eine frühe Festlegung.

**Empfehlung:** Apache-2.0 passend zum alten Repository, MIT-Teile aus Lion mit Attribution in `NOTICE`. Alternativ MIT nur nach sauberer Relizenzierung. Die Entscheidung trifft der Nutzer (D3); dieses Dokument trifft sie nicht.

### R5 – Tests, CI und Prüfwerkzeuge fehlen (P0)

**Evidenz**

- Die alte `test-pyinit.sh` (337 Zeilen) deckt Argumente, Guards, Contract-Fehler, App/CLI/lib, Zsh-Symlink-Integration sowie `.refactor/report.md`, `just refactor` (dirty tree) und `just merge` (explizit, Auto-Erkennung, Ambiguität) ab; im neuen Repository existiert sie nicht.
- Kein `tests/`-Verzeichnis, keine CI-Konfiguration, kein Link-Checker, kein Shellcheck-Lauf.

**Auswirkung:** Regressionen an `pyinit` werden nicht erkannt. Es gibt keinen Nachweis, dass ein frischer Checkout funktioniert. Genau die Fehler aus R1–R3 wären mit der alten Suite und einem Link-Check sofort sichtbar geworden.

**Empfehlung:** Alte Suite in Etappe 1 übernehmen und an die neue Struktur anpassen, in schnelle und langsame Fälle trennen. CI mit Link-Checker, Shellcheck, schnellen Tests und einem Fresh-Clone-Test ohne `ignore/`. Details in [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md) (D14).

### R6 – `AGENTS.md`: Contract statt Feature-Tabelle (P1)

**Evidenz**

- `lupysta/scripts/luxuspythonstack.sh:766-788` – die erzeugte `AGENTS.md` enthält den vollständigen `stack-contract-v1`-Block.
- Erzeugt wird nur der Hinweis „Generated once by `pyinit` from `stack-contract-v1`“; eine `lupysta_version` fehlt.
- Ein Feature-Stand und ein Statusbefehl existieren nicht.

**Auswirkung:** Der eingebettete Contract driftet gegen den Skill, sobald sich Workflow-Regeln ändern. Agenten können nicht unterscheiden, welche Features im Projekt tatsächlich vorhanden sind. Die Etappe-2-Abnahme (Feature-Tabelle) ist mit dem Ist-Stand unvereinbar.

**Empfehlung:** Profil, Feature-Tabelle, Herkunft (`lupysta_version`, `contract_version`) und lokale Regeln erzeugen. Den vollständigen Contract im Skill belassen. `just lupysta-status` prüft die deklarierten Einstiegspunkte, ohne etwas zu reparieren (D8, D10).

### R7 – Feature-Entfernung ohne Besitzregeln (P1)

**Evidenz**

- `lupysta/scripts/luxuspythonstack.sh:445-449` – gemeinsame Ignore-Einträge für alle Features.
- `lupysta/scripts/luxuspythonstack.sh:552` – `just clean` löscht sämtliche Artefakte in einem Rezept.
- `pyproject.toml` (Marker, `addopts`, Dependency-Groups) und `Justfile` werden als jeweils eine Datei erzeugt.

**Auswirkung:** Beim Entfernen eines Features besteht die Gefahr, gemeinsame Infrastruktur zu löschen oder Marker und `addopts` für verbleibende Suiten zu zerstören. Die Roadmap fordert Entfernbarkeit, ohne den Besitz zu regeln.

**Empfehlung:** Besitzmatrix und Entfernungscheckliste in [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md): gemeinsame Dateien nur teilweise bereinigen, Feature-Blöcke in `Justfile`, `.gitignore`, `pyproject.toml` und Doku klar abgrenzen, entfernte Features als `removed` sichtbar lassen.

### R8 – Refactor, Merge, Worktree, Baselines und Session-Dateien (P1)

**Evidenz**

- `lupysta/scripts/luxuspythonstack.sh:601-615` – `just refactor` legt den Branch im aktuellen Checkout an (`git switch -c`), nicht in einem Worktree.
- `lupysta/scripts/luxuspythonstack.sh:616-624` – `just merge` verlangt, dass der aktuelle Branch `main` ist.
- `lupysta/scripts/luxuspythonstack.sh:633-655` – Auto-Erkennung sucht zuerst einen Branch in einem anderen Worktree, sonst den einzigen nicht gemergten Branch.
- `lupysta/scripts/luxuspythonstack.sh:679-681` – nach dem Merge wird das Worktree entfernt und der Branch gelöscht.
- `lupysta/scripts/luxuspythonstack.sh:552` – `just clean` entfernt `.refactor/` einschließlich Baselines.
- `lupysta/scripts/luxuspythonstack.sh:448-449,556` – `SESSION.md` und `JOURNAL.md` sind gitignored und werden von `clean` geschützt, liegen aber pro Worktree.
- `lupysta/SKILL.md:150-151,188-189` – beschreibt den Ablauf als Worktree-Merge.

**Auswirkung:** Nach `just refactor` ist `just merge` im selben Checkout nicht aufrufbar; der dokumentierte Ablauf funktioniert nur über die Fallback-Erkennung oder einen manuell angelegten Worktree. Session- und Journal-Dateien aus einem entfernten Worktree sind verloren, weil sie nicht versioniert sind. `clean` löscht die Vergleichsgrundlage.

**Empfehlung:** Entweder `refactor` erzeugt selbst einen Worktree, oder `merge` akzeptiert jeden Checkout und ermittelt den Branch explizit. Vor dem Entfernen eines Worktrees `SESSION.md`/`JOURNAL.md` in den Haupt-Checkout übernehmen. `clean` schützt `.refactor/baselines/`. Merge-Rolle und Verantwortung für die Übergabe dokumentieren (D5).

### R9 – E2E/Agent: Marker allein genügen nicht (P1)

**Evidenz**

- `lupysta/references/e2e-testing.md:3` – E2E sei „not part of the `pyinit` scaffold“.
- `lupysta/references/e2e-testing.md:34-41` – empfiehlt Marker plus `addopts = ["-ra", "-q", "-m", "not e2e"]`.

**Auswirkung:** `-m` filtert erst nach der Collection; Modulebene-Importe von `docker`, `httpx` oder Provider-SDKs brechen den normalen Lauf, wenn die optionalen Gruppen nicht installiert sind. Ohne Testdateien liefert pytest Exitcode 5. Ohne Preflight entstehen unklare Fehler statt klarer Meldungen.

**Empfehlung:** Root-`tests/conftest.py` mit env-gesteuertem `collect_ignore_glob`/`pytest_ignore_collect` (`LUPYSTA_E2E=1`, `LUPYSTA_AGENT=1`). Keine optionalen Importe auf Modulebene. Runner prüfen zuerst auf `test_*.py` und melden den Leerfall mit Exitcode 0; Exitcode 5 ist nur im echten Leerfall zulässig (D7).

### R10 – Reproduzierbarkeit und Voraussetzungen (P1)

**Evidenz**

- `lupysta/scripts/luxuspythonstack.sh:234` – `uv add --dev ruff pytest pytest-cov basedpyright ...` ohne Versionsangaben; die Auflösung hängt vom Zeitpunkt und vom Netzwerk ab.
- Keine dokumentierten Mindestversionen für `uv`, `just`, `git`, `bash`, `direnv`, Docker oder Provider.
- `layout_uv`/`direnv` werden implizit vorausgesetzt.

**Auswirkung:** Zwei Läufe am selben Commit können unterschiedliche Projekte erzeugen. Fehlerbilder sind nicht reproduzierbar; Voraussetzungen fallen erst zur Laufzeit auf.

**Empfehlung:** Getestete Versionen dokumentieren und in CI pinnen, Voraussetzungen je Etappe nennen, Netzwerkbedarf ausweisen, regelmäßigen CI-Lauf etablieren (D11).

### R11 – Alter und neuer Skill gleichzeitig installiert (P1)

**Evidenz**

- `lupysta/SKILL.md:2` – Name `luxuspythonstack`.
- README: „`lupysta` is the intended skill name“.
- Beide Namen können im selben Harness-Verzeichnis liegen; die Rezepte und Pfade unterscheiden sich.

**Auswirkung:** Agenten laden die falsche Version, erhalten widersprüchliche Anleitungen und verweisen auf nicht vorhandene Dateien. Die Fehlersuche ist unnötig schwer.

**Empfehlung:** Migration und Uninstall dokumentieren: alten Skill entlinken, nicht beide installieren, Installationsprüfung (`command -v pyinit`, Skill-Pfad) beschreiben (D12).

### R12 – Agent-Harness ist unterschätzt (P2)

**Evidenz**

- Lion-Harness: `tests/agentic/agent.py` ca. 2.437 Zeilen, `suite.py` ca. 362, `conftest.py` ca. 136, `ledger.py` 33, dazu szenariospezifische Fixtures.
- Die Roadmap beschreibt Etappe 5 als „kleinen nutzbaren Kern“ ohne Größenbudget.

**Auswirkung:** Die Etappe kann die veranschlagte Zeit deutlich überschreiten; ein vollständiger Port wäre ein eigenes Projekt.

**Empfehlung:** Minimaler Kern mit Richtwert ≤600 Zeilen, CLI/HTTP zuerst, MCP optional, Fake-Provider für Limits und Fehler. Go/No-Go nach Etappe 4 (D6).

### R13 – Design-Report zu groß für Etappe 3 (P2)

**Evidenz**

- Lions `scripts/design_report.py` ca. 1.524 Zeilen.
- Der vorhandene `rate-design`-Block (`lupysta/scripts/luxuspythonstack.sh:571-598`) schreibt nur Radon-/Vulture-Berichte; `save`/`compare` und Baselines fehlen.

**Auswirkung:** Die Etappe-3-Abnahme (Messung, Baseline, Vergleich) ist mit dem Ist-Stand nicht erfüllt, ein vollständiger Port wäre unverhältnismäßig.

**Empfehlung:** Schlanker Report mit `run`, `save <name>`, `compare <name>`, `.refactor/`-Layout und `incomplete`-Marker, Richtwert 200–300 Zeilen; Vollversion erst bei Bedarf (D9).

### R14 – Etappenfolge (P2)

**Evidenz**

- `PROJECT_ROADMAP_de.md:92-105` – Etappe 6 enthält das Kopieren der Test-READMEs aus `resources/`, Feature-Entfernungstests und die Integrationsmatrix.
- `resources/pyinit/` ist laut Etappe 1 die Vorlagenablage; E2E- und Agent-Vorlagen entstehen erst in Etappe 4/5.

**Auswirkung:** Etappe 6 kann nicht beginnen, bevor 4/5 abgeschlossen sind; die Etappe wird zum Sammelbecken und verliert ihre Abgrenzung.

**Empfehlung:** Vorlagenablage und Feature-Grundlagen nach Etappe 1/2 ziehen, Feature-READMEs jeweils mit der zugehörigen Suite liefern, Etappe 6 auf Entfernungstests, Integrationsmatrix, Doku-Abgleich und Release begrenzen.

### R15 – Dokumentationswidersprüche (P2)

**Evidenz**

- `lupysta/references/e2e-testing.md:3` gegen Roadmap-Etappe 4 (E2E als Grundgerüst).
- `lupysta/SKILL.md:175` – MCPHub als Voraussetzung für Agentenarbeit.
- `lupysta/SKILL.md:226-228` – `../refactoring/SKILL.md`, `../../base-system/`, `../../ai-stack/`.
- `lupysta/references/daily-commands.md:3` – Verweis außerhalb des Repositories.
- `README.md:74` – erwähnt `ignore/`, das lokal und nicht erforderlich ist.

**Auswirkung:** Menschen und Agenten erhalten widersprüchliche Anweisungen; Links sind tot. Die Sprachregelung und die Abgrenzung „Skill vs. Projekt“ verwässern.

**Empfehlung:** Referenzen neu schreiben (`lupysta-setup.md`, `e2e-testing.md`, `e2e-agent-testing.md`, `rate-and-refactor.md`), `daily-commands.md` auflösen. Externe Inhalte nur als optionale Referenz kennzeichnen, nie als Pflichtabhängigkeit. `ignore/` ausschließlich als lokale, nicht erforderliche Ablage erwähnen.

## Empfehlung Lean-MVP

1. **Etappe 1 vollständig abschließen** – Contract und Vorlagen im Skill, Metadaten, Portabilität oder Linux-only, transaktionale Initialisierung, übernommene Tests, CI, Lizenz.
2. **Etappe 2 vollständig abschließen** – kurze `SKILL.md`, Rezepte, Feature-Tabelle, `lupysta-status`, aufgelöste Referenzen, Voraussetzungen.
3. **Etappe 3 und 4 umsetzen** – schlanker Design-Report, konsistenter Refactor-/Merge-Ablauf, E2E-Grundgerüst mit Collection-Gate und Null-Test-Semantik.
4. **Etappe 5 nur mit Go/No-Go** – Entscheidung nach Etappe 4 anhand des tatsächlichen Aufwands; minimaler Kern, kein Lion-Port.
5. **Etappe 6 als Stabilisierung** – Entfernungstests, Integrationsmatrix, Doku-Abgleich, Release-Checkliste.

Nicht empfohlen: ein vollständiger Lion-Port, ein Rezeptmanager oder automatische Feature-Erkennung vor dem ersten eigenständigen Release.

## Verwandte Dokumente

- [`PROJECT_ROADMAP_de.md`](PROJECT_ROADMAP_de.md) – Etappen, Abnahmekriterien, Risiken.
- [`PROJECT_DECISIONS_de.md`](PROJECT_DECISIONS_de.md) – Entscheidungslog D1–D15 mit Status.
- [`PROJECT_FEATURES_de.md`](PROJECT_FEATURES_de.md) – Feature-Modell, Besitz, Entfernung, `lupysta-status`.
- [`PROJECT_TESTING_de.md`](PROJECT_TESTING_de.md) – Test- und CI-Strategie, Akzeptanzmatrix.
- [`PROJECTNOTE_de.md`](PROJECTNOTE_de.md) – ursprüngliche Projektnotiz.
