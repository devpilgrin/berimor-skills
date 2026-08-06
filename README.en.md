<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · **[English](README.en.md)** · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

</div>

# berimor-skills

Behavior packages (**skills**) for [berimor](https://github.com/devpilgrin/berimor) —
a deterministic agentic CLI. A skill is a declarative package: what it can do,
when to apply it, which tools are allowed. Trigger-based skill selection is done
by code, not by the model (berimor's architectural principle: decisions are
deterministic code, models are executors).

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

Rules:

- `tools` is a **ceiling, not an extension**: a skill cannot obtain a tool
  that the session doesn't have; the intersection is computed by
  deterministic code.
- `triggers` — exact phrases and slash commands; matching is done by code,
  not by the model.
- There are no secrets in a skill and never will be — masking at the
  boundaries is mandatory.

## Installation

For now: clone into `~/.config/berimor/skills/` (or `.berimor/skills/`
in the project root — project skills take precedence over global ones).
The loader is under development (see the ROADMAP in the main repository);
the format is stable.

## Example

See [`skills/code-review-ru/`](skills/code-review-ru/) — the reference skill.

## License

[Apache-2.0](LICENSE). By publishing a skill, you agree to distribute it
under the same terms.
