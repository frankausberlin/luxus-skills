> **Deutsche Arbeitsfassung.** Übersetzung von [e2e-testing.md](e2e-testing.md). Sie ist ab jetzt die gemeinsame Arbeitsgrundlage; die englische Fassung wird erst nach Abschluss der deutschen Fassung nachgezogen.

# End-to-End-Tests (optional)

Dies ist ein **optionales Muster**, nicht Teil des `pyinit`-Gerüsts. Die meisten
Projekte werden durch Unit-Tests abgedeckt, die externe Werkzeuge mocken. Echte
End-to-End-Tests nur hinzufügen, wenn der installierte Befehl gegen echte externe
Werkzeuge, das Dateisystem oder Paketmanager ausgeführt werden muss — und niemals
auf einer Workstation.

Das folgende Muster ist das des Referenzprojekts (LION); die Namen an das eigene
Projekt anpassen.

## Wann verwenden

- Das Verhalten hängt von einem echten externen Werkzeug (`apt`, `dpkg`, `lspci`, …)
  oder von Betriebssystemzustand ab, der unsicher oder unmöglich originalgetreu
  zu mocken ist.
- Du möchtest die paketierte/installierte ausführbare Datei prüfen, nicht den
  importierten Quellcode.
- Ein Container kann jeden Seiteneffekt isolieren, sodass der Host unberührt bleibt.

Wenn ein gemockter Test das Verhalten beweisen kann, den gemockten Test bevorzugen.
End-to-End-Tests sind langsamer, benötigen Docker und Netzwerkzugriff und sind
schwieriger deterministisch zu halten.

## Aufbau der Einrichtung

Die Opt-in-Mechanik aus dem Standard-Testlauf heraushalten:

```text
scripts/test-e2e.sh          # Runner: Image bauen, Container ausführen, Artefakte einsammeln
tests/e2e/
├── conftest.py              # Guards + gemeinsame Fixtures
├── Dockerfile               # das isolierte Test-Image
└── test_*.py                # Szenarien, jeweils mit pytest.mark.e2e markiert
```

Marker registrieren und standardmäßig ausschließen, damit `just test` und
`just check` niemals den Container ausführen:

```toml
[tool.pytest.ini_options]
markers = ["e2e: end-to-end tests that run in a disposable Docker container"]
addopts = ["-ra", "-q", "-m", "not e2e"]
```

Ein eigenes Rezept hinzufügen, statt E2E in `just check` einzubinden:

```make
# Echter CLI-/Paket-Lebenszyklus in einem Wegwerf-Docker-Container
test-e2e:
    bash scripts/test-e2e.sh
```

## Isolationsregeln

- **Wegwerf-Container, keine Host-Mounts.** Die Tests in einem frischen Container
  ausführen, der aus einem Projekt-Image erstellt wurde; nichts außerhalb des
  Containers darf sich ändern.
- **Kein privilegierter Modus, kein Host-Netzwerk**, es sei denn, ein Szenario
  benötigt ausdrücklich einen Download zur Bauzeit. Keine Laufzeit-Netzwerkverbindung
  aktivieren, um einen Fehler zu beheben.
- **Temporärer beschreibbarer Zustand.** Ein temporäres `HOME`, `XDG_*` und
  Arbeitsverzeichnis im Container verwenden, damit die Umgebung des Entwicklers
  niemals gelesen wird.
- **Ausdrückliches Opt-in.** Der Runner setzt eine Marker-Umgebungsvariable
  (zum Beispiel `APP_E2E_CONTAINER=1`). Die gemeinsame `conftest.py` überspringt
  die Suite, wenn dieser Marker fehlt, damit ein versehentliches `pytest` kein
  echtes System verändern kann.
- **Root nur im Container.** Wenn ein Szenario Paketänderungen benötigt, läuft es
  als Root im Wegwerf-Container und wechselt für den Rest zu einem unprivilegierten
  Benutzer. Das Projekt selbst benötigt niemals Root.

## Aufgaben des Runners

Ein gutes `scripts/test-e2e.sh`:

1. Erstellt ein eindeutiges, von Git ignoriertes Verzeichnis `e2e-artifacts.*` für die
   Diagnose.
2. Baut das Image aus dem `Dockerfile` und der Lock-Datei (BuildKit für Cache-Mounts)
   und installiert Abhängigkeiten vor dem Quellcode, damit reine Codeänderungen
   schnell neu gebaut werden.
3. Startet einen frischen Container mit dem Opt-in-Marker und ohne Host-Mounts.
4. Führt `pytest tests/e2e -m e2e` mit einem JUnit-Bericht im Artefaktverzeichnis
   aus.
5. Kopiert die Artefakte auf den Host und entfernt den Container beim Beenden,
   auch nach Fehlern.

## Ein Szenario hinzufügen

- Das Modul mit `pytestmark = pytest.mark.e2e` markieren; die gemeinsame
  `conftest.py` wendet den Guard an und stellt die Subprozess- und
  Umgebungs-Fixtures bereit.
- Den **installierten** Befehl als Subprozess aufrufen, nicht das importierte Modul.
- Fixtures eigenständig und reihenfolgeunabhängig halten; synthetische Secrets
  verwenden.
- Exit-Codes, Ausgabe, persistierten Zustand und Berechtigungen prüfen — und dass
  schreibgeschützte Befehle nichts schreiben.
- Jeden Subprozess-Befehl, Exit-Code und Ausgabe im Artefaktverzeichnis
  aufzeichnen, damit Fehler diagnostizierbar sind.
- Die genau ausgeführten Befehle und etwaige Abdeckungsgrenzen berichten. Zuerst
  das gewöhnliche Qualitätsgate ausführen, dann den E2E-Runner.

## Weiterführende Dokumentation

- [`../SKILL_de.md`](../SKILL_de.md) — der Stack-Workflow und die `just`-Rezepte
  (englische Fassung: [`../SKILL.md`](../SKILL.md)).
- [`../../../luxuspythonstack.md`](../../../luxuspythonstack.md) — vollständige
  Stack-Dokumentation.
