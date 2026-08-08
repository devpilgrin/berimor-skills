<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · **[日本語](README.ja.md)** · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-skills

[berimor](https://github.com/devpilgrin/berimor) — 決定論的な agentic CLI —
のための振る舞いパッケージ（**スキル**）。スキルは宣言的なパッケージです：
何ができるか、いつ適用するか、どのツールが許可されるか。トリガーによる
スキルの選択は、モデルではなくコードが行います（berimor のアーキテクチャ
原則：意思決定は決定論的コード、モデルは実行者）。

## フォーマット

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

ルール：

- `tools` は**拡張ではなく上限**です：スキルはセッションに存在しない
  ツールを取得できません。積集合は決定論的コードが計算します。
- `triggers` — 完全一致のフレーズとスラッシュコマンド。マッチングは
  モデルではなくコードが行います。
- スキルにシークレットは存在せず、今後も存在しません — 境界での
  マスキングは必須です。

## インストール

現時点では：`~/.config/berimor/skills/` にクローンしてください
（またはプロジェクトルートの `.berimor/skills/` — プロジェクトの
スキルはグローバルより優先されます）。ローダーは開発中です
（メインリポジトリの ROADMAP を参照）。フォーマットは安定しています。

## 例

[`skills/code-review-ru/`](skills/code-review-ru/) を参照 —
リファレンススキルです。

## ライセンス

[Apache-2.0](LICENSE)。スキルを公開することで、同じ条件で配布することに
同意したものとみなされます。
