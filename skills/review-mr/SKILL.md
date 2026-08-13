---
name: review-mr
description: Автоматическое ревью GitLab MR с анализом кода и публикацией комментариев
invoke_by: user
---

# GitLab MR Review Skill

Автоматическое ревью Merge Request'ов в GitLab с комплексным анализом кода и интерактивной публикацией комментариев.

## Требуемые разрешения

Для работы skill требует доступ к следующим командам (добавьте в `~/.claude/settings.json`):

```json
{
  "allow": [
    "Bash(glab *)",
    "Bash(git diff *)",
    "Bash(git log *)",
    "Bash(git show *)"
  ]
}
```

Команда `claude-tools review-setup` добавит их автоматически.

Вместо прямого вызова `glab` можно использовать обёртку `dp glab` (тот же синтаксис команд) — если она доступна в окружении, используй её.

## Использование

```
/review-mr <gitlab-mr-url>
```

**Примеры:**
```
/review-mr https://gitlab.ru/platforms/distribution/-/merge_requests/179
/review-mr https://gitlab.ru/platforms/airflow/-/merge_requests/42
```

## Что делает skill

1. **Проверяет наличие GitLab токена** (только при первом запуске)
2. **Получает diff из MR** через glab CLI
3. **Анализирует изменения** по комплексу политик ревью
4. **Показывает результаты** в CLI с подробными замечаниями
5. **Спрашивает**, что публиковать: всё, конкретные комментарии, или отменить
6. **Публикует выбранные комментарии** в GitLab MR

## Настройка (первый запуск)

Перед первым использованием skill необходимо настроить GitLab токен:

```bash
claude-tools review-setup
```

Эта команда:
1. Запросит GitLab Personal Access Token
2. Укажет какие права должны быть у токена (api)
3. Настроит glab CLI для работы с gitlab.ru
4. Сохранит конфигурацию

После выполнения `review-setup` skill `/review-mr` сразу будет работать.

### Создание токена

Токен нужно создать на: https://gitlab.ru/-/user_settings/personal_access_tokens

**Обязательное право (scope):**
- ✅ `api` - полный доступ к API

Это право уже включает чтение репозиториев, создание комментариев, и всё необходимое для работы skill.

## Инструкции для Claude

**ВАЖНО:**
- Все комментарии и описания должны быть **на русском языке**
- После анализа **ОБЯЗАТЕЛЬНО** используй AskUserQuestion для интерактивного выбора комментариев
- НЕ публикуй комментарии автоматически без подтверждения пользователя

### Шаг 1: Проверка GitLab токена

Проверь статус glab аутентификации:

```bash
glab auth status 2>&1 | grep -q "Logged in to gitlab.ru"
if [ $? -eq 0 ]; then
  echo "✓ GitLab authentication configured"
else
  echo "⚠ GitLab authentication not configured"
  exit 1
fi
```

Если не настроен, подскажи пользователю:

> ⚠ GitLab не настроен. Выполните настройку:
>
> ```bash
> claude-tools review-setup
> ```
>
> Эта команда настроит glab с вашим токеном для gitlab.ru.

### Шаг 2: Парсинг MR URL

Из URL вида `https://gitlab.ru/group/project/-/merge_requests/123` извлеки:
- **project**: `group/project`
- **MR number**: `123`

### Шаг 3: Получение MR информации

```bash
glab mr view <MR_NUMBER> --repo <PROJECT> --output json
```

Получи:
- Название MR
- Source branch
- Target branch
- Автор
- Статус (open/merged/closed)

### Шаг 4: Получение diff

```bash
glab mr diff <MR_NUMBER> --repo <PROJECT>
```

Проанализируй diff и определи:
- Какие файлы изменены
- Язык программирования (Python, Go, Kotlin, etc.)
- Тип изменений (новый код, рефакторинг, fix, tests)

### Шаг 5: Анализ по политикам

Используй политики из `./policies/` (в той же директории что и SKILL.md):

- **python-review-policy.md** - для Python кода
- **security-policy.md** - для всех языков (безопасность)
- **tests-policy.md** - для тестового кода
- **general-policy.md** - общие правила

Для КАЖДОГО замечания сформируй структуру **на русском языке**:

```
[SEVERITY] Category: Rule ID
File: path/to/file.py:42
Описание: Подробное описание проблемы на русском

Предложение: Как исправить

Code:
```python
<проблемный код>
```

Fixed:
```python
<исправленный вариант>
```
```

**Severity levels:**
- 🔴 CRITICAL - критичные уязвимости, баги
- 🟡 MAJOR - серьезные проблемы, требуют исправления
- 🔵 MINOR - мелкие улучшения
- ⚪ NIT - стилистические замечания

### Шаг 6: Показать результаты в CLI

Выведи красиво отформатированный отчет **на русском языке**:

```
=== Ревью GitLab MR ===
MR: #123 "Добавить новую функцию"
Автор: username
Изменено файлов: 5 (3 Python, 2 теста)

=== Результаты ревью ===
Найдено замечаний: 12
  🔴 CRITICAL: 1
  🟡 MAJOR: 3
  🔵 MINOR: 5
  ⚪ NIT: 3

---

🔴 [CRITICAL] Security: SEC-1
File: src/api/handler.py:42
SQL injection уязвимость в сыром запросе

Code:
```python
query = f"SELECT * FROM users WHERE id = {user_id}"
```

Предложение: Используйте параметризованные запросы
Fixed:
```python
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))
```

---

[... остальные замечания ...]

```

### Шаг 7: Интерактивный выбор публикации

**ОБЯЗАТЕЛЬНО**: После показа результатов используй AskUserQuestion для выбора комментариев. НЕ пропускай этот шаг!

Сначала спроси основной выбор:

```python
AskUserQuestion([{
    "question": "Что сделать с результатами ревью?",
    "header": "Действие",
    "multiSelect": false,
    "options": [
        {
            "label": "Опубликовать все замечания",
            "description": f"Опубликовать все {total_issues} комментариев в GitLab MR"
        },
        {
            "label": "Выбрать конкретные замечания (Recommended)",
            "description": "Выбрать какие комментарии опубликовать через форму множественного выбора"
        },
        {
            "label": "Не публиковать",
            "description": "Только посмотреть результаты, ничего не публиковать"
        }
    ]
}])
```

Если выбрано "Выбрать конкретные", используй AskUserQuestion с **multiSelect: true**:

```python
# Сформируй options из всех замечаний
options = []
for i, issue in enumerate(issues, 1):
    severity_emoji = {"CRITICAL": "🔴", "MAJOR": "🟡", "MINOR": "🔵", "NIT": "⚪"}[issue.severity]
    options.append({
        "label": f"{severity_emoji} {issue.severity} - {issue.file}:{issue.line}",
        "description": issue.short_description[:100]  # Первые 100 символов
    })

AskUserQuestion([{
    "question": "Выберите замечания для публикации (можно выбрать несколько):",
    "header": "Комментарии",
    "multiSelect": true,  # ВАЖНО: множественный выбор!
    "options": options
}])
```

Результат будет список выбранных options, опубликуй только их.

### Шаг 8: Публикация комментариев

**ВАЖНО:** Публикуй комментарии как **inline комментарии** прямо к конкретным строкам кода через GitLab API.

#### Шаг 8.1: Получить metadata MR

Для inline комментариев нужны SHA коммитов:

```bash
# Получить project_id из URL или через glab
glab mr view <MR_NUMBER> --repo <PROJECT> --output json | jq '{project_id, iid, source_branch, target_branch}'

# Получить последнюю версию MR с SHA
glab api projects/<PROJECT_ID>/merge_requests/<MR_NUMBER>/versions | jq '.[0] | {head_commit_sha, base_commit_sha, start_commit_sha}'
```

Сохрани:
- `project_id` - ID проекта
- `head_commit_sha` - SHA головного коммита
- `base_commit_sha` - SHA базового коммита
- `start_commit_sha` - SHA начального коммита (обычно равен base_commit_sha)

#### Шаг 8.2: Создать inline комментарий

Для каждого выбранного замечания создай inline комментарий через GitLab API:

```bash
glab api -X POST projects/<PROJECT_ID>/merge_requests/<MR_NUMBER>/discussions \
  -H "Content-Type: application/json" \
  --input - <<'EOF'
{
  "body": "🔴 [CRITICAL] Security: SEC-1\n\n**Файл:** src/api/handler.py **Строка:** 42\n\n**Описание:** SQL injection уязвимость в сыром запросе. Использование f-string для формирования SQL запроса позволяет выполнить произвольный SQL код.\n\n**Предложение:** Используйте параметризованные запросы\n\n**Проблемный код:**\n```python\nquery = f\"SELECT * FROM users WHERE id = {user_id}\"\ncursor.execute(query)\n```\n\n**Исправленный вариант:**\n```python\nquery = \"SELECT * FROM users WHERE id = %s\"\ncursor.execute(query, (user_id,))\n```\n\n---\n🤖 Автоматическое ревью от Claude Code\nПолитика: security-policy.md",
  "position": {
    "position_type": "text",
    "base_sha": "<BASE_COMMIT_SHA>",
    "head_sha": "<HEAD_COMMIT_SHA>",
    "start_sha": "<START_COMMIT_SHA>",
    "new_path": "src/api/handler.py",
    "old_path": "src/api/handler.py",
    "new_line": 42
  }
}
EOF
```

**Параметры position:**
- `position_type`: всегда `"text"`
- `base_sha`, `head_sha`, `start_sha`: SHA из шага 8.1
- `new_path` и `old_path`: путь к файлу (обычно одинаковые)
- `new_line`: номер строки в новой версии файла для новых/измененных строк
- `old_line`: номер строки в старой версии для удаленных строк (если комментируешь удаленную строку, используй `old_line` вместо `new_line`)

**Определение new_line из diff:**
- Парсируй вывод `glab mr diff <MR_NUMBER>`
- Найди строку с проблемой в diff
- Посчитай номер строки из заголовка `@@` и смещения

**Важно:**
- Все комментарии **на русском языке**
- Используй emoji для severity: 🔴 CRITICAL, 🟡 MAJOR, 🔵 MINOR, ⚪ NIT
- Всегда показывай проблемный и исправленный код с подсветкой синтаксиса
- Оставляй ссылку на политику откуда взято правило
- Комментарии появятся **inline прямо на строках кода** в diff MR

### Шаг 9: Финальный отчет

Покажи итоговый отчет на русском:

```
✅ Опубликовано 3 из 12 замечаний как inline комментарии

🔴 CRITICAL: 1 комментарий опубликован
🟡 MAJOR: 2 комментария опубликовано

Комментарии появились в diff на конкретных строках кода.
Просмотреть MR: https://gitlab.ru/.../-/merge_requests/123
```

## Политики ревью

Все политики находятся в `./policies/`:

- **python-review-policy.md** - Python best practices, Airflow, data processing
- **kotlin-review-policy.md** - Kotlin idioms, coroutines, null safety
- **go-review-policy.md** - Go idioms, concurrency, error handling
- **sql-review-policy.md** - SQL queries, schema design, performance, ORM
- **security-policy.md** - Security vulnerabilities (OWASP Top 10, CWE)
- **tests-policy.md** - Test quality and coverage (pytest best practices)
- **general-policy.md** - General code quality (Clean Code, Code Smells)

При анализе применяй ВСЕ подходящие политики к коду.

### Источники политик

Политики основаны на проверенных индустриальных стандартах:

- **Python**: PEP 8, Python docs, pylint/flake8/mypy rules
- **Security**: OWASP Top 10, CWE (Common Weakness Enumeration), Bandit
- **Tests**: pytest docs, TDD/Testing Best Practices
- **General**: "Clean Code" (Robert Martin), "Refactoring" (Martin Fowler)
- **Airflow**: Airflow Official Best Practices

### Кастомизация политик

Можно редактировать политики под нужды команды:

- Добавить специфичные для проекта правила
- Изменить severity для правил
- Отключить неприменимые правила (закомментировать в таблице)
- Добавить примеры из вашего кодбейза

## Дополнительные возможности

### Повторное ревью после исправлений

```
/review-mr <url> --incremental
```

Проверь только новые коммиты с момента последнего ревью.

### Ревью только измененных файлов

Автоматически игнорируй файлы, где удалено больше чем добавлено (cleanup), и файлы с изменениями < 5 строк.

### Фильтр по severity

Если замечаний очень много (>50), предложи опубликовать только CRITICAL и MAJOR.

## Требования

- `glab` CLI установлен и настроен
- GitLab Personal Access Token с правами `api`
- Доступ к GitLab по сети

## Troubleshooting

**Ошибка "glab not found":**
```bash
brew install glab
```

**Ошибка "401 Unauthorized":**
Токен истек или некорректный. Удали старый:
```bash
rm ~/.claude/.gitlab_token
```

И запусти skill заново для ввода нового токена.

**MR не найден:**
Проверь, что URL корректный и у вас есть доступ к репозиторию.

## Связанные файлы

- Политики: `.claude/skills/review-mr/policies/*.md`
- Токен: `~/.claude/.gitlab_token`
- glab конфиг: `~/Library/Application Support/glab-cli/config.yml`
