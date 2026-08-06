<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · **[Deutsch](README.de.md)** · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

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

## Installation

Derzeit: nach `~/.config/berimor/skills/` klonen (oder `.berimor/skills/`
im Projektstamm — Projekt-Skills haben Vorrang vor globalen). Der Loader
befindet sich in Entwicklung (siehe ROADMAP des Haupt-Repositorys);
das Format ist stabil.

## Beispiel

Siehe [`skills/code-review-ru/`](skills/code-review-ru/) — der Referenz-Skill.

## Lizenz

[Apache-2.0](LICENSE). Mit der Veröffentlichung eines Skills erklären Sie
sich einverstanden, ihn unter denselben Bedingungen zu verbreiten.
