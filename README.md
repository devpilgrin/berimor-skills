<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

**[Русский](README.md)** · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-skills

Пакеты поведения (**скилы**) для [berimor](https://github.com/devpilgrin/berimor) —
детерминированного agentic CLI. Скил — это декларативный пакет: что умеет,
когда применять, какие инструменты разрешены. Выбор скилла по триггерам —
кодом, не моделью (архитектурный принцип berimor: решения — детерминированный
код, модели — исполнители).

## Формат

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

Правила:

- `tools` — **потолок, не расширение**: скилл не может получить инструмент,
  которого нет у сессии; пересечение вычисляет детерминированный код.
- `triggers` — точные фразы и slash-команды; совпадение ищет код, не модель.
- Секретов в скилле нет и не будет — маскировщик на границах обязателен.

## Установка

Сейчас: клонируйте в `~/.config/berimor/skills/` (или `.berimor/skills/`
в корне проекта — проектные скилы сильнее глобальных). Загрузчик —
в разработке (см. ROADMAP основного репозитория); формат стабилен.

## Пример

См. [`skills/code-review-ru/`](skills/code-review-ru/) — эталонный скилл.

## Лицензия

[Apache-2.0](LICENSE). Публикуя скилл, вы соглашаетесь распространять его
на тех же условиях.
