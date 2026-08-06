<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · **[Español](README.es.md)** · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

</div>

# berimor-skills

Paquetes de comportamiento (**skills**) para [berimor](https://github.com/devpilgrin/berimor) —
un CLI agéntico determinista. Un skill es un paquete declarativo: qué sabe
hacer, cuándo aplicarlo, qué herramientas están permitidas. La selección del
skill mediante disparadores la realiza el código, no el modelo (principio
arquitectónico de berimor: las decisiones son código determinista, los modelos
son ejecutores).

## Formato

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

Reglas:

- `tools` es un **techo, no una ampliación**: un skill no puede obtener una
  herramienta que la sesión no tenga; la intersección la calcula el código
  determinista.
- `triggers` — frases exactas y comandos slash; la coincidencia la busca el
  código, no el modelo.
- No hay secretos en un skill ni los habrá — el enmascaramiento en los
  límites es obligatorio.

## Instalación

Por ahora: clona en `~/.config/berimor/skills/` (o `.berimor/skills/`
en la raíz del proyecto — los skills del proyecto tienen prioridad sobre
los globales). El cargador está en desarrollo (consulta la ROADMAP del
repositorio principal); el formato es estable.

## Ejemplo

Consulta [`skills/code-review-ru/`](skills/code-review-ru/) — el skill de
referencia.

## Licencia

[Apache-2.0](LICENSE). Al publicar un skill, aceptas distribuirlo en las
mismas condiciones.
