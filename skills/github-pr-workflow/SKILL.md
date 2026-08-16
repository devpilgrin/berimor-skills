---
name: github-pr-workflow
version: 0.1.0
description: Полный жизненный цикл GitHub PR — ветка, коммит, создание PR, мониторинг CI, автофикс падений, мердж; через gh или git + curl. GitHub PR lifecycle: branch, commit, open, CI, merge.
triggers:
  - "создай PR"
  - "открой пул-реквест"
  - "смерджи PR"
  - "github pr"
tools:
  - files.read
  - files.write
  - files.edit
  - terminal.exec
permissions:
  - fs-write
  - exec
model_tier: strong
---

# Жизненный цикл GitHub Pull Request

Полное руководство по управлению циклом PR. Каждый раздел показывает сначала
путь через `gh`, затем запасной вариант `git` + `curl` для машин без `gh`.

## Предусловия

- Аутентификация в GitHub: работающий `gh auth` либо токен (`GITHUB_TOKEN`)
  для REST API.
- Внутри git-репозитория с GitHub-ремоутом.

### Быстрое определение метода аутентификации

```bash
# Определи метод, который будешь использовать во всём воркфлоу
if command -v gh &>/dev/null && gh auth status &>/dev/null; then
  AUTH="gh"
else
  AUTH="git"
  # Нужен токен для API-вызовов
  if [ -z "$GITHUB_TOKEN" ]; then
    # Попробуй извлечь из git credential store
    GITHUB_TOKEN=$(printf "protocol=https\nhost=github.com\n\n" | git credential fill 2>/dev/null | sed -n 's/^password=//p')
  fi
fi
echo "Using: $AUTH"
```

Если токена нет ни в окружении, ни в credential store — попроси пользователя
настроить `gh auth login` или выдать `GITHUB_TOKEN`.

### Извлечение owner/repo из git-ремоута

Многим `curl`-командам нужен `owner/repo`. Извлеки из ремоута:

```bash
# Работает и для HTTPS, и для SSH URL
REMOTE_URL=$(git remote get-url origin)
OWNER_REPO=$(echo "$REMOTE_URL" | sed -E 's|.*github\.com[:/]||; s|\.git$||')
OWNER=$(echo "$OWNER_REPO" | cut -d/ -f1)
REPO=$(echo "$OWNER_REPO" | cut -d/ -f2)
echo "Owner: $OWNER, Repo: $REPO"
```

---

## 1. Создание ветки

Чистый `git` — одинаково в обоих случаях:

```bash
# Убедись, что актуален
git fetch origin
git checkout main && git pull origin main

# Создай и переключись на новую ветку
git checkout -b feat/add-user-authentication
```

Конвенции имён веток:

- `feat/description` — новые фичи
- `fix/description` — исправления багов
- `refactor/description` — перестройка кода
- `docs/description` — документация
- `ci/description` — изменения CI/CD

## 2. Коммиты

Вноси изменения файловыми инструментами (files.write, files.edit), затем
коммить:

```bash
# Стейдж точечных файлов
git add src/auth.py src/models/user.py tests/test_auth.py

# Коммит в формате Conventional Commits
git commit -m "feat: add JWT-based user authentication

- Add login/register endpoints
- Add User model with password hashing
- Add auth middleware for protected routes
- Add unit tests for auth flow"
```

Формат сообщения (Conventional Commits):

```
type(scope): short description

Longer explanation if needed. Wrap at 72 characters.
```

Типы: `feat`, `fix`, `refactor`, `docs`, `test`, `ci`, `chore`, `perf`

## 3. Пуш и создание PR

### Пуш ветки (одинаково в обоих случаях)

```bash
git push -u origin HEAD
```

### Создание PR

**Через gh:**

```bash
gh pr create \
  --title "feat: add JWT-based user authentication" \
  --body "## Summary
- Adds login and register API endpoints
- JWT token generation and validation

## Test Plan
- [ ] Unit tests pass

Closes #42"
```

Опции: `--draft`, `--reviewer user1,user2`, `--label "enhancement"`,
`--base develop`

**Через git + curl:**

```bash
BRANCH=$(git branch --show-current)

curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/$OWNER/$REPO/pulls \
  -d "{
    \"title\": \"feat: add JWT-based user authentication\",
    \"body\": \"## Summary\\nAdds login and register API endpoints.\\n\\nCloses #42\",
    \"head\": \"$BRANCH\",
    \"base\": \"main\"
  }"
```

JSON ответа содержит `number` PR — сохрани для дальнейших команд.

Для черновика добавь `\"draft\": true` в тело JSON.

## 4. Мониторинг статуса CI

### Проверка статуса CI

**Через gh:**

```bash
# Разовая проверка
gh pr checks

# Следить до завершения всех проверок (поллинг каждые 10s)
gh pr checks --watch
```

**Через git + curl:**

```bash
# SHA последнего коммита текущей ветки
SHA=$(git rev-parse HEAD)

# Комбинированный статус
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/commits/$SHA/status \
  | python -c "
import sys, json
data = json.load(sys.stdin)
print(f\"Overall: {data['state']}\")
for s in data.get('statuses', []):
    print(f\"  {s['context']}: {s['state']} - {s.get('description', '')}\")"

# Отдельно — check runs GitHub Actions (другой эндпоинт)
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/commits/$SHA/check-runs \
  | python -c "
import sys, json
data = json.load(sys.stdin)
for cr in data.get('check_runs', []):
    print(f\"  {cr['name']}: {cr['status']} / {cr['conclusion'] or 'pending'}\")"
```

### Поллинг до завершения (git + curl)

```bash
# Простой цикл поллинга — каждые 30 секунд, до 10 минут
SHA=$(git rev-parse HEAD)
for i in $(seq 1 20); do
  STATUS=$(curl -s \
    -H "Authorization: token $GITHUB_TOKEN" \
    https://api.github.com/repos/$OWNER/$REPO/commits/$SHA/status \
    | python -c "import sys,json; print(json.load(sys.stdin)['state'])")
  echo "Check $i: $STATUS"
  if [ "$STATUS" = "success" ] || [ "$STATUS" = "failure" ] || [ "$STATUS" = "error" ]; then
    break
  fi
  sleep 30
done
```

## 5. Автофикс падений CI

Когда CI падает — диагностируй и чини. Цикл работает с любым методом
аутентификации.

### Шаг 1: Получи детали падения

**Через gh:**

```bash
# Недавние workflow-запуски на этой ветке
gh run list --branch $(git branch --show-current) --limit 5

# Логи упавших джоб
gh run view <RUN_ID> --log-failed
```

**Через git + curl:**

```bash
BRANCH=$(git branch --show-current)

# Список workflow-запусков на ветке
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/actions/runs?branch=$BRANCH&per_page=5" \
  | python -c "
import sys, json
runs = json.load(sys.stdin)['workflow_runs']
for r in runs:
    print(f\"Run {r['id']}: {r['name']} - {r['conclusion'] or r['status']}\")"

# Логи упавших джоб (скачать zip, распаковать, прочитать)
RUN_ID=<run_id>
curl -s -L \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/runs/$RUN_ID/logs \
  -o /tmp/ci-logs.zip
cd /tmp && unzip -o ci-logs.zip -d ci-logs && cat ci-logs/*.txt
```

### Шаг 2: Исправь и запушь

Определив проблему, исправь код файловыми инструментами (files.edit,
files.write):

```bash
git add <fixed_files>
git commit -m "fix: resolve CI failure in <check_name>"
git push
```

### Шаг 3: Проверь

Перепроверь статус CI командами из раздела 4.

### Паттерн цикла автофикса

Когда просят автофикс CI, следуй циклу:

1. Проверь статус CI → определи падения.
2. Прочитай логи падения → пойми ошибку.
3. files.read + files.edit/files.write → исправь код.
4. `git add . && git commit -m "fix: ..." && git push`.
5. Дождись CI → перепроверь статус.
6. Повторяй, если всё ещё падает (до 3 попыток, затем спроси пользователя).

## 6. Мердж

**Через gh:**

```bash
# Squash-мердж + удаление ветки (чистейший вариант для feature-веток)
gh pr merge --squash --delete-branch

# Включить авто-мердж (смерджится, когда все проверки пройдут)
gh pr merge --auto --squash --delete-branch
```

**Через git + curl:**

```bash
PR_NUMBER=<number>

# Мердж PR через API (squash)
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/pulls/$PR_NUMBER/merge \
  -d "{
    \"merge_method\": \"squash\",
    \"commit_title\": \"feat: add user authentication (#$PR_NUMBER)\"
  }"

# Удали удалённую ветку после мерджа
BRANCH=$(git branch --show-current)
git push origin --delete $BRANCH

# Вернись на main локально
git checkout main && git pull origin main
git branch -d $BRANCH
```

Методы мерджа: `"merge"` (merge-коммит), `"squash"`, `"rebase"`

### Включение авто-мерджа (curl)

```bash
# Авто-мердж требует, чтобы он был включён в настройках репозитория.
# Используется GraphQL API, т.к. REST авто-мердж не поддерживает.
PR_NODE_ID=$(curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/pulls/$PR_NUMBER \
  | python -c "import sys,json; print(json.load(sys.stdin)['node_id'])")

curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/graphql \
  -d "{\"query\": \"mutation { enablePullRequestAutoMerge(input: {pullRequestId: \\\"$PR_NODE_ID\\\", mergeMethod: SQUASH}) { clientMutationId } }\"}"
```

## 7. Полный пример воркфлоу

```bash
# 1. Начни с чистого main
git checkout main && git pull origin main

# 2. Ветка
git checkout -b fix/login-redirect-bug

# 3. (Агент вносит изменения файловыми инструментами)

# 4. Коммит
git add src/auth/login.py tests/test_login.py
git commit -m "fix: correct redirect URL after login

Preserves the ?next= parameter instead of always redirecting to /dashboard."

# 5. Пуш
git push -u origin HEAD

# 6. Создай PR (gh или curl — по доступности)
# ... (см. раздел 3)

# 7. Мониторь CI (см. раздел 4)

# 8. Мердж, когда зелёный (см. раздел 6)
```

## Шпаргалка по командам PR

| Действие | gh | git + curl |
|----------|-----|-----------|
| Мои PR | `gh pr list --author @me` | `curl -s -H "Authorization: token $GITHUB_TOKEN" "https://api.github.com/repos/$OWNER/$REPO/pulls?state=open"` |
| Diff PR | `gh pr diff` | `git diff main...HEAD` (локально) или `curl -H "Accept: application/vnd.github.diff" ...` |
| Комментарий | `gh pr comment N --body "..."` | `curl -X POST .../issues/N/comments -d '{"body":"..."}'` |
| Запрос ревью | `gh pr edit N --add-reviewer user` | `curl -X POST .../pulls/N/requested_reviewers -d '{"reviewers":["user"]}'` |
| Закрыть PR | `gh pr close N` | `curl -X PATCH .../pulls/N -d '{"state":"closed"}'` |
| Чекаут чужого PR | `gh pr checkout N` | `git fetch origin pull/N/head:pr-N && git checkout pr-N` |
