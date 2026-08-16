---
name: webapp-testing
version: 1.0.0
description: "Используй этот скилл для взаимодействия с локальными веб-приложениями и их тестирования через Playwright: проверка функциональности фронтенда, отладка поведения UI, захват скриншотов браузера, просмотр логов браузера. Use for testing local web apps with Playwright: verifying frontend functionality, debugging UI behavior, capturing screenshots and browser logs."
triggers:
  - "webapp testing"
  - "playwright"
  - "тестирование веб-приложения"
  - "проверь фронтенд"
  - "скриншот страницы"
  - "browser automation"
tools:
  - files.read
  - files.write
  - terminal.exec
permissions:
  - fs-write
  - exec
license: Proprietary. LICENSE.txt has complete terms
---

# Тестирование веб-приложений

> **Примечание (berimor):** для работы этого скилла требуется playwright MCP-сервер (см. `docs/mcp-servers.md` основного репозитория).

Чтобы тестировать локальные веб-приложения, пиши нативные Python-скрипты на Playwright.

**Доступные вспомогательные скрипты**:
- `scripts/with_server.py` — управляет жизненным циклом сервера (поддерживает несколько серверов)

**Всегда сначала запускай скрипты с `--help`**, чтобы увидеть использование. НЕ читай исходный код, пока не попробуешь запустить скрипт и не убедишься, что кастомное решение абсолютно необходимо. Эти скрипты могут быть очень большими и тем самым загрязнят твоё контекстное окно. Они существуют, чтобы вызываться напрямую как скрипты-«чёрные ящики», а не для поглощения в контекстное окно. Запуск выполняй через terminal.exec.

## Дерево решений: выбор подхода

```
Задача пользователя → Это статический HTML?
    ├─ Да → Прочитай HTML-файл напрямую (files.read), чтобы определить селекторы
    │         ├─ Успех → Пиши Playwright-скрипт, используя селекторы
    │         └─ Неудача/неполно → Обращайся как с динамическим (ниже)
    │
    └─ Нет (динамическое веб-приложение) → Сервер уже запущен?
        ├─ Нет → Запусти: python scripts/with_server.py --help
        │        Затем используй хелпер + пиши упрощённый Playwright-скрипт
        │
        └─ Да → Паттерн «разведка-затем-действие»:
            1. Перейди на страницу и дождись networkidle
            2. Сделай скриншот или инспектируй DOM
            3. Определи селекторы по отрендеренному состоянию
            4. Выполняй действия с обнаруженными селекторами
```

## Пример: использование with_server.py

Чтобы запустить сервер, сначала выполни `--help`, затем используй хелпер:

**Один сервер:**
```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**Несколько серверов (например, backend + frontend):**
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

Чтобы создать скрипт автоматизации, включай только логику Playwright (серверы управляются автоматически):
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True) # Always launch chromium in headless mode
    page = browser.new_page()
    page.goto('http://localhost:5173') # Server already running and ready
    page.wait_for_load_state('networkidle') # CRITICAL: Wait for JS to execute
    # ... your automation logic
    browser.close()
```

## Паттерн «разведка-затем-действие»

1. **Инспектируй отрендеренный DOM**:
   ```python
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()
   ```

2. **Определи селекторы** по результатам инспекции

3. **Выполняй действия**, используя обнаруженные селекторы

## Частая ловушка

❌ **Не** инспектируй DOM до ожидания `networkidle` на динамических приложениях
✅ **Всегда** жди `page.wait_for_load_state('networkidle')` перед инспекцией

## Лучшие практики

- **Используй встроенные скрипты как чёрные ящики** — чтобы выполнить задачу, подумай, может ли помочь один из скриптов в `scripts/`. Эти скрипты надёжно обрабатывают типовые сложные рабочие процессы, не загромождая контекстное окно. Используй `--help`, чтобы увидеть использование, затем вызывай напрямую.
- Используй `sync_playwright()` для синхронных скриптов
- Всегда закрывай браузер по завершении
- Используй описательные селекторы: `text=`, `role=`, CSS-селекторы или ID
- Добавляй подходящие ожидания: `page.wait_for_selector()` или `page.wait_for_timeout()`

## Справочные файлы

- **examples/** — примеры, показывающие типовые паттерны:
  - `element_discovery.py` — обнаружение кнопок, ссылок и полей ввода на странице
  - `static_html_automation.py` — использование file:// URL для локального HTML
  - `console_logging.py` — захват консольных логов во время автоматизации
