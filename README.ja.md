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

## カタログ内容

**コード関連：**

| スキル | 説明 |
|---|---|
| `code-review-ru` | diff のレビュー：スタイル、エラー、セキュリティリスク |
| `security-audit-ru` | diff/ファイルへの迅速なセキュリティチェック — OWASP クラスの問題 |
| `bug-triage-ru` | バグのトリアージ：再現 → 原因の特定 → ピンポイント修正 |
| `test-writer-ru` | 既存コードに対する挙動ベースのテスト、エッジケース込み |
| `commit-msg-ru` | staged diff からのコミットメッセージ：何を・なぜ、言い換えではなく |

**ドキュメント**（anthropics/skills から適応；スクリプトはユーザーのマシンで実行されます）：

| スキル | 説明 |
|---|---|
| `docx` | Word 文書（.docx/.dotx）：作成、読み取り、編集、tracked changes |
| `xlsx` | スプレッドシート（.xlsx/.csv/.tsv）：数式、書式設定、データクリーニング |
| `pptx` | プレゼンテーション（.pptx/.potx）：デッキの作成と編集 |
| `pdf` | PDF：読み取りと抽出、結合/分割、フォーム、スキャンの OCR |

**エージェントエコシステム：**

| スキル | 説明 |
|---|---|
| `mcp-builder` | MCP サーバーの構築（Python FastMCP / Node SDK）— ツール設計からパッケージングまで |
| `skill-creator` | スキルの作成と改善、効果測定（evals） |

**コミュニケーションとデザイン：**

| スキル | 説明 |
|---|---|
| `doc-coauthoring` | ドキュメントの共同執筆：ユーザーとの構造化された反復 |
| `internal-comms` | 社内コミュニケーション：ステータスレポート、3P アップデート、FAQ、インシデント |
| `frontend-design` | 意図的な UI ビジュアルデザイン：美学、タイポグラフィ、定型化なし |

## インストール

カタログはデフォルトで berimor に組み込まれています：チャットからは
`/skills add`（ピッカーがこのリポジトリの利用可能な項目を表示）、
CLI からは `berimor skill install <name>`
（例：`berimor skill install code-review-ru`）。その他のソース：`berimor skill install <name> --from <git-url>`、または `~/.config/berimor/skills/`（グローバル）
もしくは `.berimor/skills/`（プロジェクト、グローバルより優先）に
クローン。カタログで利用可能な一覧：`berimor skill list --available`。

## 例

[`skills/code-review-ru/`](skills/code-review-ru/) を参照 —
リファレンススキルです。

## ライセンス

[Apache-2.0](LICENSE)。スキルを公開することで、同じ条件で配布することに
同意したものとみなされます。
