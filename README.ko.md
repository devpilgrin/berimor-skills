<div align="center">

<img src="assets/logo.png" alt="Berimor" width="640">

[Русский](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · [Français](README.fr.md) · [Español](README.es.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · **[한국어](README.ko.md)**

[![Socket](https://badge.socket.dev/npm/package/berimor)](https://socket.dev/npm/package/berimor)

</div>

# berimor-skills

[berimor](https://github.com/devpilgrin/berimor) — 결정론적 에이전틱 CLI —
를 위한 동작 패키지(**스킬**). 스킬은 선언적 패키지입니다: 무엇을 할 수
있는지, 언제 적용하는지, 어떤 도구가 허용되는지. 트리거 기반 스킬 선택은
모델이 아니라 코드가 수행합니다(berimor의 아키텍처 원칙: 결정은 결정론적
코드, 모델은 실행자).

## 형식

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

규칙:

- `tools`는 **확장이 아니라 상한**입니다: 스킬은 세션에 없는 도구를 얻을
  수 없으며, 교집합은 결정론적 코드가 계산합니다.
- `triggers` — 정확한 구문과 슬래시 명령; 일치 여부는 모델이 아니라
  코드가 찾습니다.
- 스킬에는 시크릿이 없으며 앞으로도 없을 것입니다 — 경계에서의 마스킹은
  필수입니다.

## 설치

현재: `~/.config/berimor/skills/`에 클론하세요(또는 프로젝트 루트의
`.berimor/skills/` — 프로젝트 스킬이 전역 스킬보다 우선합니다). 로더는
개발 중입니다(메인 리포지토리의 ROADMAP 참조); 형식은 안정적입니다.

## 예시

[`skills/code-review-ru/`](skills/code-review-ru/) 참조 — 레퍼런스
스킬입니다.

## 라이선스

[Apache-2.0](LICENSE). 스킬을 게시하면 동일한 조건으로 배포하는 데
동의하는 것입니다.
