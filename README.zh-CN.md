<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · **[简体中文](README.zh-CN.md)** · [日本語](README.ja.md) · [한국어](README.ko.md)

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-skills

[berimor](https://github.com/devpilgrin/berimor) 的行为包（**skill**）——
一个确定性的 agentic CLI。skill 是一个声明式包：它能做什么、何时应用、
允许使用哪些工具。基于触发器的 skill 选择由代码完成，而非模型
（berimor 的架构原则：决策是确定性代码，模型是执行者）。

## 格式

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

规则：

- `tools` 是**上限，而非扩展**：skill 无法获得会话中不存在的工具；
  交集由确定性代码计算。
- `triggers` —— 精确短语和斜杠命令；匹配由代码查找，而非模型。
- skill 中不存在秘密，也永远不会存在 —— 边界处的掩码处理是强制的。

## 目录内容

**代码相关：**

| Skill | 描述 |
|---|---|
| `code-review-ru` | diff 评审：风格、错误、安全风险 |
| `security-audit-ru` | 对 diff/文件进行快速安全检查 — OWASP 类问题 |
| `bug-triage-ru` | bug 分诊：复现 → 定位原因 → 精准修复 |
| `test-writer-ru` | 按行为为现有代码编写测试，含边界情况 |
| `commit-msg-ru` | 根据 staged diff 生成提交信息：说明做了什么和为什么，而非复述 |

**文档**（改编自 anthropics/skills；脚本在用户机器上执行）：

| Skill | 描述 |
|---|---|
| `docx` | Word 文档（.docx/.dotx）：创建、读取、编辑、tracked changes |
| `xlsx` | 电子表格（.xlsx/.csv/.tsv）：公式、格式化、数据清洗 |
| `pptx` | 演示文稿（.pptx/.potx）：创建和编辑幻灯片 |
| `pdf` | PDF：读取与提取、合并/拆分、表单、扫描件 OCR |

**代理生态：**

| Skill | 描述 |
|---|---|
| `mcp-builder` | 构建 MCP 服务器（Python FastMCP / Node SDK）—— 从工具设计到打包 |
| `skill-creator` | 创建和改进 skill，衡量其效果（evals） |

**沟通与设计：**

| Skill | 描述 |
|---|---|
| `doc-coauthoring` | 文档协作：与用户进行结构化迭代 |
| `internal-comms` | 内部沟通：状态报告、3P 更新、FAQ、事故 |
| `frontend-design` | 有意识的 UI 视觉设计：美学、排版、拒绝模板化 |

**创意与 QA**（改编自 anthropics/skills，echelon-2）：

| Skill | 描述 |
|---|---|
| `canvas-design` | 海报与封面（PNG/PDF）：构图、配色、内置字体 |
| `brand-guidelines` | 用于制品和文档的品牌配色与排版 |
| `theme-factory` | 10 套现成设计主题：文档、幻灯片、落地页、报告 |
| `algorithmic-art` | 生成艺术：带种子（seeded）的 p5.js 草图 + 交互式查看器 |
| `webapp-testing` | 通过 Playwright 对 Web 应用进行 E2E 检查（需要 playwright MCP 服务器，参见 berimor 的 docs/mcp-servers.md） |

## 安装

目录默认内置在 berimor 中：在聊天中 —— `/skills add`（选择器会显示
此仓库中的可用项），在 CLI 中 —— `berimor skill install <name>`
（例如 `berimor skill install code-review-ru`）。任何其他来源：
`berimor skill install <name> --from <git-url>`，或克隆到
`~/.config/berimor/skills/`（全局）或 `.berimor/skills/`（项目级，
优先于全局）。查看目录中的可用项：`berimor skill list --available`。

## 示例

参见 [`skills/code-review-ru/`](skills/code-review-ru/) —— 参考 skill。

## 许可证

[Apache-2.0](LICENSE)。发布 skill 即表示您同意以相同条款进行分发。
