<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · **[English](README.en.md)** · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

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

## Catalog contents

**Working with code:**

| Skill | Description |
|---|---|
| `code-review-ru` | Diff review: style, bugs, security risks |
| `security-audit-ru` | Quick security pass over a diff/file — OWASP problem classes |
| `bug-triage-ru` | Bug triage: reproduction → root-cause localization → targeted fix |
| `test-writer-ru` | Behavior-based tests for existing code, with edge cases |
| `commit-msg-ru` | Commit message from the staged diff: what and why, not a retelling |

**Documents** (adapted from anthropics/skills; scripts run on the user's machine):

| Skill | Description |
|---|---|
| `docx` | Word documents (.docx/.dotx): create, read, edit, tracked changes |
| `xlsx` | Spreadsheets (.xlsx/.csv/.tsv): formulas, formatting, data cleanup |
| `pptx` | Presentations (.pptx/.potx): creating and editing decks |
| `pdf` | PDF: reading and extraction, merge/split, forms, OCR of scans |

**Agent ecosystem:**

| Skill | Description |
|---|---|
| `mcp-builder` | Building MCP servers (Python FastMCP / Node SDK) — from tool design to packaging |
| `skill-creator` | Creating and improving skills, measuring their effectiveness (evals) |

**Communication and design:**

| Skill | Description |
|---|---|
| `doc-coauthoring` | Documentation co-authoring: structured iterations with the user |
| `internal-comms` | Internal communications: status reports, 3P updates, FAQs, incidents |
| `frontend-design` | Deliberate visual UI design: aesthetics, typography, no cookie-cutter output |

**Creative and QA** (adapted from anthropics/skills, echelon-2):

| Skill | Description |
|---|---|
| `canvas-design` | Posters and covers (PNG/PDF): composition, palette, bundled fonts |
| `brand-guidelines` | Brand colors and typography for artifacts and documents |
| `theme-factory` | 10 ready-made design themes: documents, slides, landing pages, reports |
| `algorithmic-art` | Generative art: seeded p5.js sketches + interactive viewer |
| `webapp-testing` | E2E testing of web apps via Playwright (requires the playwright MCP server, see docs/mcp-servers.md in berimor) |

## Installation

The catalog is built into berimor by default: from chat — `/skills add`
(a picker will show what's available from this repository), from the CLI —
`berimor skill install <name>` (e.g. `berimor skill install code-review-ru`).
Any other source: `berimor skill install <name> --from <git-url>` or cloning
into `~/.config/berimor/skills/` (global) or `.berimor/skills/` (project,
takes precedence over global). List what's available in the catalog:
`berimor skill list --available`.

## Example

See [`skills/code-review-ru/`](skills/code-review-ru/) — the reference skill.

## License

[Apache-2.0](LICENSE). By publishing a skill, you agree to distribute it
under the same terms.
