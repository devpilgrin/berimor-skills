<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · **[Français](README.fr.md)** · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-skills

Paquets de comportement (**skills**) pour [berimor](https://github.com/devpilgrin/berimor) —
un CLI agentique déterministe. Un skill est un paquet déclaratif : ce qu'il
sait faire, quand l'appliquer, quels outils sont autorisés. La sélection d'un
skill par déclencheurs est effectuée par du code, pas par le modèle (principe
architectural de berimor : les décisions sont du code déterministe, les modèles
sont des exécutants).

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

Règles :

- `tools` est un **plafond, pas une extension** : un skill ne peut pas obtenir
  un outil que la session ne possède pas ; l'intersection est calculée par du
  code déterministe.
- `triggers` — phrases exactes et commandes slash ; la correspondance est
  recherchée par le code, pas par le modèle.
- Il n'y a pas de secrets dans un skill et il n'y en aura jamais — le masquage
  aux frontières est obligatoire.

## Contenu du catalogue

**Travail sur le code :**

| Skill | Description |
|---|---|
| `code-review-ru` | Revue de diffs : style, erreurs, risques de sécurité |
| `security-audit-ru` | Passage sécurité rapide sur diff/fichier — classes de problèmes OWASP |
| `bug-triage-ru` | Triage d'un bug : reproduction → localisation de la cause → correctif ciblé |
| `test-writer-ru` | Tests pour du code existant basés sur le comportement, avec cas limites |
| `commit-msg-ru` | Message de commit d'après le diff stagé : quoi et pourquoi, pas une paraphrase |

**Documents** (adapté de anthropics/skills ; les scripts s'exécutent sur la machine de l'utilisateur) :

| Skill | Description |
|---|---|
| `docx` | Documents Word (.docx/.dotx) : création, lecture, modification, tracked changes |
| `xlsx` | Feuilles de calcul (.xlsx/.csv/.tsv) : formules, mise en forme, nettoyage de données |
| `pptx` | Présentations (.pptx/.potx) : création et édition de decks |
| `pdf` | PDF : lecture et extraction, fusion/division, formulaires, OCR de scans |

**Écosystème de l'agent :**

| Skill | Description |
|---|---|
| `mcp-builder` | Construction de serveurs MCP (Python FastMCP / Node SDK) — de la conception des outils au packaging |
| `skill-creator` | Création et amélioration de skills, mesure de leur efficacité (evals) |

**Communication et design :**

| Skill | Description |
|---|---|
| `doc-coauthoring` | Co-rédaction de documentation : itérations structurées avec l'utilisateur |
| `internal-comms` | Communications internes : rapports de statut, mises à jour 3P, FAQ, incidents |
| `frontend-design` | Design visuel réfléchi d'UI : esthétique, typographie, sans templates |

**Créatif et QA** (adapté de anthropics/skills, echelon-2) :

| Skill | Description |
|---|---|
| `canvas-design` | Affiches et couvertures (PNG/PDF) : composition, palette, polices incluses |
| `brand-guidelines` | Couleurs de marque et typographie pour les artefacts et documents |
| `theme-factory` | 10 thèmes de conception prêts à l'emploi : documents, diapositives, landing pages, rapports |
| `algorithmic-art` | Art génératif : sketches p5.js avec seed + visionneuse interactive |
| `webapp-testing` | Vérification E2E d'applications web via Playwright (nécessite le serveur MCP playwright, voir docs/mcp-servers.md dans berimor) |

**Nécessite berimor 0.32.0+** : les skills déclarent des `permissions` (net/exec/fs-write/spawn) cohérentes avec le plafond `tools` — `berimor skill lint` et l'installation depuis le catalogue l'imposent en fail-closed. Avant publication : `berimor skill lint <chemin>` et, en option, `berimor skill review <chemin>`.

## Installation

Le catalogue par défaut est intégré à berimor : depuis le chat —
`/skills add` (le sélecteur affichera les skills disponibles de ce dépôt),
depuis le CLI — `berimor skill install <name>` (par exemple
`berimor skill install code-review-ru`). Toute autre source :
`berimor skill install <name> --from <git-url>` ou clonage dans
`~/.config/berimor/skills/` (global) ou `.berimor/skills/` (projet,
prioritaire sur les globaux). Liste des skills disponibles du catalogue :
`berimor skill list --available`.

## Exemple

Voir [`skills/code-review-ru/`](skills/code-review-ru/) — le skill de référence.

## Licence

[Apache-2.0](LICENSE). En publiant un skill, vous acceptez de le distribuer
aux mêmes conditions.
