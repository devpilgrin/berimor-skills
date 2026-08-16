---
name: github-issues
version: 0.1.0
description: Создание, поиск, триаж, метки, назначение и закрытие GitHub-ишью — через gh или REST API curl. Create, triage, label, assign GitHub issues via gh or REST.
triggers:
  - "создай issue"
  - "github issue"
  - "затриажь ишью"
  - "закрой issue"
tools:
  - terminal.exec
permissions:
  - exec
model_tier: weak
---

# Управление GitHub Issues

Создание, поиск, триаж и управление GitHub-ишью. Каждый раздел показывает
сначала `gh`, затем запасной вариант через `curl`.

## Предусловия

- Аутентификация в GitHub: работающий `gh auth` либо токен (`GITHUB_TOKEN`)
  для REST API.
- Внутри git-репозитория с GitHub-ремоутом, либо репозиторий указан явно.

### Настройка

```bash
if command -v gh &>/dev/null && gh auth status &>/dev/null; then
  AUTH="gh"
else
  AUTH="git"
  if [ -z "$GITHUB_TOKEN" ]; then
    # Попробуй извлечь из git credential store
    GITHUB_TOKEN=$(printf "protocol=https\nhost=github.com\n\n" | git credential fill 2>/dev/null | sed -n 's/^password=//p')
  fi
fi

REMOTE_URL=$(git remote get-url origin)
OWNER_REPO=$(echo "$REMOTE_URL" | sed -E 's|.*github\.com[:/]||; s|\.git$||')
OWNER=$(echo "$OWNER_REPO" | cut -d/ -f1)
REPO=$(echo "$OWNER_REPO" | cut -d/ -f2)
```

Если токена нет ни в окружении, ни в credential store — попроси пользователя
настроить `gh auth login` или выдать `GITHUB_TOKEN`.

---

## 1. Просмотр ишью

**Через gh:**

```bash
gh issue list
gh issue list --state open --label "bug"
gh issue list --assignee @me
gh issue list --search "authentication error" --state all
gh issue view 42
```

**Через curl:**

```bash
# Список открытых ишью
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?state=open&per_page=20" \
  | python -c "
import sys, json
for i in json.load(sys.stdin):
    if 'pull_request' not in i:  # API GitHub возвращает и PR в /issues
        labels = ', '.join(l['name'] for l in i['labels'])
        print(f\"#{i['number']:5}  {i['state']:6}  {labels:30}  {i['title']}\")"

# Фильтр по метке
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?state=open&labels=bug&per_page=20" \
  | python -c "
import sys, json
for i in json.load(sys.stdin):
    if 'pull_request' not in i:
        print(f\"#{i['number']}  {i['title']}\")"

# Конкретный ишью
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42 \
  | python -c "
import sys, json
i = json.load(sys.stdin)
labels = ', '.join(l['name'] for l in i['labels'])
assignees = ', '.join(a['login'] for a in i['assignees'])
print(f\"#{i['number']}: {i['title']}\")
print(f\"State: {i['state']}  Labels: {labels}  Assignees: {assignees}\")
print(f\"Author: {i['user']['login']}  Created: {i['created_at']}\")
print(f\"\n{i['body']}\")"

# Поиск ишью
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/issues?q=authentication+error+repo:$OWNER/$REPO" \
  | python -c "
import sys, json
for i in json.load(sys.stdin)['items']:
    print(f\"#{i['number']}  {i['state']:6}  {i['title']}\")"
```

## 2. Создание ишью

**Через gh:**

```bash
gh issue create \
  --title "Login redirect ignores ?next= parameter" \
  --body "## Description
After logging in, users always land on /dashboard.

## Steps to Reproduce
1. Navigate to /settings while logged out
2. Get redirected to /login?next=/settings
3. Log in
4. Actual: redirected to /dashboard (should go to /settings)

## Expected Behavior
Respect the ?next= query parameter." \
  --label "bug,backend" \
  --assignee "username"
```

**Через curl:**

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues \
  -d '{
    "title": "Login redirect ignores ?next= parameter",
    "body": "## Description\nAfter logging in, users always land on /dashboard.\n\n## Steps to Reproduce\n1. Navigate to /settings while logged out\n2. Get redirected to /login?next=/settings\n3. Log in\n4. Actual: redirected to /dashboard\n\n## Expected Behavior\nRespect the ?next= query parameter.",
    "labels": ["bug", "backend"],
    "assignees": ["username"]
  }'
```

### Шаблон баг-репорта

```
## Bug Description
<Что происходит>

## Steps to Reproduce
1. <шаг>
2. <шаг>

## Expected Behavior
<Что должно происходить>

## Actual Behavior
<Что происходит на самом деле>

## Environment
- OS: <os>
- Version: <version>
```

### Шаблон фича-реквеста

```
## Feature Description
<Что нужно>

## Motivation
<Зачем это полезно>

## Proposed Solution
<Как это могло бы работать>

## Alternatives Considered
<Другие подходы>
```

## 3. Управление ишью

### Добавление/снятие меток

**Через gh:**

```bash
gh issue edit 42 --add-label "priority:high,bug"
gh issue edit 42 --remove-label "needs-triage"
```

**Через curl:**

```bash
# Добавить метки
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42/labels \
  -d '{"labels": ["priority:high", "bug"]}'

# Снять метку
curl -s -X DELETE \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42/labels/needs-triage

# Список доступных меток репозитория
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/labels \
  | python -c "
import sys, json
for l in json.load(sys.stdin):
    print(f\"  {l['name']:30}  {l.get('description', '')}\")"
```

### Назначение

**Через gh:**

```bash
gh issue edit 42 --add-assignee username
gh issue edit 42 --add-assignee @me
```

**Через curl:**

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42/assignees \
  -d '{"assignees": ["username"]}'
```

### Комментирование

**Через gh:**

```bash
gh issue comment 42 --body "Investigated — root cause is in auth middleware. Working on a fix."
```

**Через curl:**

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42/comments \
  -d '{"body": "Investigated — root cause is in auth middleware. Working on a fix."}'
```

### Закрытие и переоткрытие

**Через gh:**

```bash
gh issue close 42
gh issue close 42 --reason "not planned"
gh issue reopen 42
```

**Через curl:**

```bash
# Закрыть
curl -s -X PATCH \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42 \
  -d '{"state": "closed", "state_reason": "completed"}'

# Переоткрыть
curl -s -X PATCH \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42 \
  -d '{"state": "open"}'
```

### Связывание ишью с PR

Ишью закрываются автоматически при мердже PR с правильными ключевыми словами
в теле:

```
Closes #42
Fixes #42
Resolves #42
```

Создать ветку от ишью:

**Через gh:**

```bash
gh issue develop 42 --checkout
```

**Через git (ручной эквивалент):**

```bash
git checkout main && git pull origin main
git checkout -b fix/issue-42-login-redirect
```

## 4. Воркфлоу триажа ишью

Когда просят затриажить ишью:

1. **Список неразобранных ишью:**

```bash
# Через gh
gh issue list --label "needs-triage" --state open

# Через curl
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?labels=needs-triage&state=open" \
  | python -c "
import sys, json
for i in json.load(sys.stdin):
    if 'pull_request' not in i:
        print(f\"#{i['number']}  {i['title']}\")"
```

2. **Прочитай и категоризируй** каждый ишью (смотри детали, пойми баг/фичу).
3. **Проставь метки и приоритет** (см. «Управление ишью» выше).
4. **Назначь**, если владелец очевиден.
5. **Прокомментируй заметками триажа**, если нужно.

## 5. Массовые операции

Для пакетных операций комбинируй API-вызовы с шелл-скриптингом:

**Через gh:**

```bash
# Закрыть все ишью с конкретной меткой
gh issue list --label "wontfix" --json number --jq '.[].number' | \
  xargs -I {} gh issue close {} --reason "not planned"
```

**Через curl:**

```bash
# Список номеров ишью с меткой, затем закрытие каждого
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?labels=wontfix&state=open" \
  | python -c "import sys,json; [print(i['number']) for i in json.load(sys.stdin)]" \
  | while read num; do
    curl -s -X PATCH \
      -H "Authorization: token $GITHUB_TOKEN" \
      https://api.github.com/repos/$OWNER/$REPO/issues/$num \
      -d '{"state": "closed", "state_reason": "not_planned"}'
    echo "Closed #$num"
  done
```

## Шпаргалка

| Действие | gh | curl-эндпоинт |
|----------|-----|---------------|
| Список ишью | `gh issue list` | `GET /repos/{o}/{r}/issues` |
| Просмотр ишью | `gh issue view N` | `GET /repos/{o}/{r}/issues/N` |
| Создать ишью | `gh issue create ...` | `POST /repos/{o}/{r}/issues` |
| Добавить метки | `gh issue edit N --add-label ...` | `POST /repos/{o}/{r}/issues/N/labels` |
| Назначить | `gh issue edit N --add-assignee ...` | `POST /repos/{o}/{r}/issues/N/assignees` |
| Комментарий | `gh issue comment N --body ...` | `POST /repos/{o}/{r}/issues/N/comments` |
| Закрыть | `gh issue close N` | `PATCH /repos/{o}/{r}/issues/N` |
| Поиск | `gh issue list --search "..."` | `GET /search/issues?q=...` |
