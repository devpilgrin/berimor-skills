<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · **[简体中文](README.zh-CN.md)** · [日本語](README.ja.md) · [한국어](README.ko.md)

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

## 安装

目前：克隆到 `~/.config/berimor/skills/`（或项目根目录下的
`.berimor/skills/` —— 项目级 skill 优先于全局 skill）。加载器正在
开发中（参见主仓库的 ROADMAP）；格式已稳定。

## 示例

参见 [`skills/code-review-ru/`](skills/code-review-ru/) —— 参考 skill。

## 许可证

[Apache-2.0](LICENSE)。发布 skill 即表示您同意以相同条款进行分发。
