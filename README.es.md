<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · **[Español](README.es.md)** · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

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

## Contenido del catálogo

**Trabajo con código:**

| Skill | Descripción |
|---|---|
| `code-review-ru` | Revisión de diffs: estilo, errores, riesgos de seguridad |
| `security-audit-ru` | Pasada rápida de seguridad sobre diff/archivo — clases de problemas OWASP |
| `bug-triage-ru` | Triaje de un bug: reproducción → localización de la causa → corrección puntual |
| `test-writer-ru` | Tests para código existente según el comportamiento, con casos límite |
| `commit-msg-ru` | Mensaje de commit a partir del diff en staging: qué y por qué, no un resumen |

**Documentos** (adaptado de anthropics/skills; los scripts se ejecutan en la máquina del usuario):

| Skill | Descripción |
|---|---|
| `docx` | Documentos Word (.docx/.dotx): creación, lectura, edición, tracked changes |
| `xlsx` | Hojas de cálculo (.xlsx/.csv/.tsv): fórmulas, formato, limpieza de datos |
| `pptx` | Presentaciones (.pptx/.potx): creación y edición de decks |
| `pdf` | PDF: lectura y extracción, unir/dividir, formularios, OCR de escaneos |

**Ecosistema del agente:**

| Skill | Descripción |
|---|---|
| `mcp-builder` | Construcción de servidores MCP (Python FastMCP / Node SDK) — del diseño de herramientas al empaquetado |
| `skill-creator` | Creación y mejora de skills, medición de su eficacia (evals) |

**Comunicación y diseño:**

| Skill | Descripción |
|---|---|
| `doc-coauthoring` | Coautoría de documentación: iteraciones estructuradas con el usuario |
| `internal-comms` | Comunicaciones internas: informes de estado, actualizaciones 3P, FAQ, incidentes |
| `frontend-design` | Diseño visual consciente de UI: estética, tipografía, sin plantillas genéricas |

**Procesos de ingeniería** (adaptados de NousResearch/hermes-agent, escalón-3):

| Skill | Descripción |
|---|---|
| `systematic-debugging` | Depuración desde la causa raíz: reproducción, aislamiento, corrección probada |
| `test-driven-development` | Ciclo RED-GREEN-REFACTOR: pruebas antes del código, verificadas con ejecución real |
| `plan` | Modo planificación: plan accionable en `.berimor/plans/` antes de implementar |
| `spike` | Experimento puntual para validar una idea antes de construir |
| `simplify-code` | Limpieza de cambios recientes: duplicados, ramas redundantes, código muerto |
| `requesting-code-review` | Solicitud de revisión de cambios: diff, contexto, criterios de aceptación |
| `github-pr-workflow` | Ciclo de vida de PR vía gh: rama, commit, apertura, CI, correcciones |
| `github-issues` | Trabajo con issues vía gh: creación, triaje, etiquetas, asignación |

**Creatividad y QA** (adaptado de anthropics/skills, echelon-2):

| Skill | Descripción |
|---|---|
| `canvas-design` | Pósters y portadas (PNG/PDF): composición, paleta, fuentes incluidas |
| `brand-guidelines` | Colores de marca y tipografía para artefactos y documentos |
| `theme-factory` | 10 temas de diseño listos: documentos, diapositivas, landing pages, informes |
| `algorithmic-art` | Arte generativo: sketches de p5.js con seed + visor interactivo |
| `webapp-testing` | Verificación E2E de aplicaciones web mediante Playwright (requiere el servidor MCP playwright, ver docs/mcp-servers.md en berimor) |

**Requiere berimor 0.32.0+**: los skills declaran `permissions` (net/exec/fs-write/spawn) coherentes con el techo `tools` — `berimor skill lint` y la instalación desde el catálogo lo exigen en fail-closed. Antes de publicar: `berimor skill lint <ruta>` y, opcionalmente, `berimor skill review <ruta>`.

## Instalación

El catálogo predeterminado está integrado en berimor: desde el chat —
`/skills add` (el selector mostrará los skills disponibles de este
repositorio), desde el CLI — `berimor skill install <name>` (por ejemplo
`berimor skill install code-review-ru`). Cualquier otra fuente:
`berimor skill install <name> --from <git-url>` o clonación en
`~/.config/berimor/skills/` (global) o `.berimor/skills/` (proyecto,
con prioridad sobre los globales). Lista de los skills disponibles del
catálogo: `berimor skill list --available`.

## Ejemplo

Consulta [`skills/code-review-ru/`](skills/code-review-ru/) — el skill de
referencia.

## Licencia

[Apache-2.0](LICENSE). Al publicar un skill, aceptas distribuirlo en las
mismas condiciones.
