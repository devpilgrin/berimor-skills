<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · **[Deutsch](README.de.md)** · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-skills

Verhaltenspakete (**Skills**) für [berimor](https://github.com/devpilgrin/berimor) —
eine deterministische agentische CLI. Ein Skill ist ein deklaratives Paket:
was er kann, wann er anzuwenden ist, welche Werkzeuge erlaubt sind. Die Auswahl
eines Skills anhand von Triggern erfolgt durch Code, nicht durch das Modell
(Architekturprinzip von berimor: Entscheidungen sind deterministischer Code,
Modelle sind Ausführende).

## Format

```
skills/<name>/
├── SKILL.md        # YAML frontmatter + тело (инструкции модели)
└── scripts/        # опционально: вспомогательные скрипты скилла
```

### SKILL.md

````markdown
---
name: code-review-ru
version: 0.1.0
description: Ревью diff'ов на русском — стиль, ошибки, риски
triggers:
  - "проверь код"
  - "ревью"
  - "/review"
tools:              # потолок инструментов скилла (пересечение с гейтом)
  - files.read
  - files.list
  - terminal.exec
model_tier: strong  # минимальный класс модели (weak|strong)
---

# Инструкции для модели

(Тело — системный контекст, который получает модель, когда скилл активен.)
````

Regeln:

- `tools` ist eine **Obergrenze, keine Erweiterung**: Ein Skill kann kein
  Werkzeug erhalten, das die Sitzung nicht hat; die Schnittmenge berechnet
  deterministischer Code.
- `triggers` — exakte Phrasen und Slash-Befehle; die Übereinstimmung sucht
  der Code, nicht das Modell.
- Secrets gibt es im Skill nicht und wird es nie geben — die Maskierung an
  den Grenzen ist obligatorisch.

## Kataloginhalt

**Arbeit mit Code:**

| Skill | Beschreibung |
|---|---|
| `code-review-ru` | Review von Diffs: Stil, Fehler, Sicherheitsrisiken |
| `security-audit-ru` | Schneller Security-Durchgang über Diff/Datei — OWASP-Problemklassen |
| `bug-triage-ru` | Bug-Triage: Reproduktion → Ursachenlokalisierung → gezielter Fix |
| `test-writer-ru` | Verhaltensbasierte Tests für bestehenden Code, mit Grenzfällen |
| `commit-msg-ru` | Commit-Nachricht aus dem Staged-Diff: was und warum, keine Nacherzählung |

**Dokumente** (adaptiert aus anthropics/skills; Skripte laufen auf der Maschine des Benutzers):

| Skill | Beschreibung |
|---|---|
| `docx` | Word-Dokumente (.docx/.dotx): Erstellen, Lesen, Bearbeiten, tracked changes |
| `xlsx` | Tabellenkalkulationen (.xlsx/.csv/.tsv): Formeln, Formatierung, Datenbereinigung |
| `pptx` | Präsentationen (.pptx/.potx): Erstellen und Bearbeiten von Decks |
| `pdf` | PDF: Lesen und Extrahieren, Zusammenführen/Teilen, Formulare, OCR von Scans |

**Agenten-Ökosystem:**

| Skill | Beschreibung |
|---|---|
| `mcp-builder` | Aufbau von MCP-Servern (Python FastMCP / Node SDK) — vom Tool-Design bis zur Paketierung |
| `skill-creator` | Erstellen und Verbessern von Skills, Messung ihrer Effektivität (Evals) |

**Kommunikation und Design:**

| Skill | Beschreibung |
|---|---|
| `doc-coauthoring` | Co-Autorenschaft von Dokumentation: strukturierte Iterationen mit dem Benutzer |
| `internal-comms` | Interne Kommunikation: Statusberichte, 3P-Updates, FAQs, Incidents |
| `frontend-design` | Bewusstes visuelles UI-Design: Ästhetik, Typografie, ohne Schablonenhaftigkeit |

## Installation

Der Katalog ist standardmäßig in berimor eingebaut: aus dem Chat — `/skills add`
(ein Picker zeigt das Verfügbare aus diesem Repository), aus der CLI —
`berimor skill install <name>` (z. B. `berimor skill install code-review-ru`).
Jede andere Quelle: `berimor skill install <name> --from <git-url>` oder Klonen
nach `~/.config/berimor/skills/` (global) bzw. `.berimor/skills/` (Projekt,
hat Vorrang vor globalen). Liste des im Katalog Verfügbaren:
`berimor skill list --available`.

## Beispiel

Siehe [`skills/code-review-ru/`](skills/code-review-ru/) — der Referenz-Skill.

## Lizenz

[Apache-2.0](LICENSE). Mit der Veröffentlichung eines Skills erklären Sie
sich einverstanden, ihn unter denselben Bedingungen zu verbreiten.
