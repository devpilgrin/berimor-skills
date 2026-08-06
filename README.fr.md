<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · **[Français](README.fr.md)** · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

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

## Installation

Pour l'instant : clonez dans `~/.config/berimor/skills/` (ou `.berimor/skills/`
à la racine du projet — les skills de projet sont prioritaires sur les skills
globaux). Le chargeur est en cours de développement (voir la ROADMAP du dépôt
principal) ; le format est stable.

## Exemple

Voir [`skills/code-review-ru/`](skills/code-review-ru/) — le skill de référence.

## Licence

[Apache-2.0](LICENSE). En publiant un skill, vous acceptez de le distribuer
aux mêmes conditions.
