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

## 카탈로그 내용

**코드 작업:**

| 스킬 | 설명 |
|---|---|
| `code-review-ru` | diff 리뷰: 스타일, 오류, 보안 리스크 |
| `security-audit-ru` | diff/파일에 대한 빠른 보안 점검 — OWASP 클래스 문제 |
| `bug-triage-ru` | 버그 트리아지: 재현 → 원인 파악 → 정밀 수정 |
| `test-writer-ru` | 기존 코드에 대한 동작 기반 테스트, 경계 사례 포함 |
| `commit-msg-ru` | staged diff 기반 커밋 메시지: 무엇을, 왜 — 단순 반복 아님 |

**문서** (anthropics/skills에서 각색; 스크립트는 사용자 머신에서 실행됩니다):

| 스킬 | 설명 |
|---|---|
| `docx` | Word 문서 (.docx/.dotx): 생성, 읽기, 편집, tracked changes |
| `xlsx` | 스프레드시트 (.xlsx/.csv/.tsv): 수식, 서식, 데이터 정리 |
| `pptx` | 프레젠테이션 (.pptx/.potx): 덱 생성 및 편집 |
| `pdf` | PDF: 읽기 및 추출, 병합/분할, 폼, 스캔 OCR |

**에이전트 생태계:**

| 스킬 | 설명 |
|---|---|
| `mcp-builder` | MCP 서버 구축 (Python FastMCP / Node SDK) — 도구 설계부터 패키징까지 |
| `skill-creator` | 스킬 생성 및 개선, 효과 측정 (evals) |

**커뮤니케이션과 디자인:**

| 스킬 | 설명 |
|---|---|
| `doc-coauthoring` | 문서 공동 작성: 사용자와의 구조화된 반복 |
| `internal-comms` | 내부 커뮤니케이션: 상태 보고서, 3P 업데이트, FAQ, 인시던트 |
| `frontend-design` | 의도적인 UI 비주얼 디자인: 미학, 타이포그래피, 템플릿 없음 |

**크리에이티브와 QA** (anthropics/skills에서 각색, echelon-2):

| 스킬 | 설명 |
|---|---|
| `canvas-design` | 포스터와 커버 (PNG/PDF): 구성, 팔레트, 번들 폰트 |
| `brand-guidelines` | 아티팩트와 문서를 위한 브랜드 컬러와 타이포그래피 |
| `theme-factory` | 10가지 기성 디자인 테마: 문서, 슬라이드, 랜딩 페이지, 보고서 |
| `algorithmic-art` | 제너레이티브 아트: 시드 기반 p5.js 스케치 + 인터랙티브 뷰어 |
| `webapp-testing` | Playwright를 통한 웹 애플리케이션 E2E 검증 (playwright MCP 서버 필요, berimor의 docs/mcp-servers.md 참조) |

## 설치

카탈로그는 기본적으로 berimor에 내장되어 있습니다: 채팅에서는
`/skills add`(피커가 이 리포지토리의 사용 가능한 항목을 표시),
CLI에서는 `berimor skill install <name>`
(예: `berimor skill install code-review-ru`). 다른 소스: `berimor skill install <name> --from <git-url>` 또는 `~/.config/berimor/skills/`(전역) 또는
`.berimor/skills/`(프로젝트, 전역보다 우선)에 클론. 카탈로그에서
사용 가능한 목록: `berimor skill list --available`.

## 예시

[`skills/code-review-ru/`](skills/code-review-ru/) 참조 — 레퍼런스
스킬입니다.

## 라이선스

[Apache-2.0](LICENSE). 스킬을 게시하면 동일한 조건으로 배포하는 데
동의하는 것입니다.
