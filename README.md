> **Deutsche Arbeitsfassung.** Übersetzung von [README.md](README.md). Sie ist ab jetzt die gemeinsame Arbeitsgrundlage; die englische Fassung wird erst nach Abschluss der deutschen Fassung nachgezogen.

(das repo heißt im moment noch luxus-skills, das wird noch umbenannt in 'ludesta')

# LUDESTA - Luxus Development Stack

Wiederverwendbare Agent-Skills für einen komfortablen, konsistenten Entwicklungsworkflow.

Das *luxoriös* steht für die verwendung komfortabler mechaniken und tools im entwicklungsprozess. Die idee ist, wenn ich einem agenten (=skill+harness) eine umfangreiche und komfortable projektstruktur vorgebe (init) und ihn anweise, diese effektiv einzusetzen (skill), führt dies zur verbesserung der qualität der agentischen arbeit. Vor allem durch automatismen in der ci/cd wird ein hohes maß an stabilität erreicht. 

Ein Luxus Stack (z.b. Python) befindet sich im gleichnamigen folder und besteht aus drei komponenten:
|Komponente|Beschreibung|
|-|-|
|[Stack Dokumentation](Luxus-Python-Stack/Luxus-Python-Stack.md)| Beschreibung des Stacks und seiner einrichtung<br>• Setupanleitung für alle sprachrelevanten tools und umgebungen<br>• Einrichtung der Systemumgebung, ZSH (mit p10k) und integration der stack features<br>• Beschreibung zusätzlicher Stackfeatures, die über den skill hinaus gehen.|
|[Stack Skill](Luxus-Python-Stack/lupysta/SKILL_de.md)| Beschreibung agentischen arbeitens<br>• verwendung der CI/CD<br>• abgleich der dokumentation<br>• Test und refactoring|
|[Stack Script](Luxus-Python-Stack/luxuspythonstack.sh)| Die init-funktion und weiteren stack-spezifischen funktionen<br>• init-funktion zur erzeugung eines vorkonfigurierten projektgerüstes<br>• zusätzliche stack-features außerhalb agentischer zuständigkeit<br>• vorkonfigurierte aliase zum aufruf der script-funktionen|

## Installation

Befolge einfach die Setup-Anleitung des Stacks.

## Wichtiger Hinweis

> ⚠️ Die einrichtung eines Luxus Stacks verwendet **nala** und **zsh**.

