# GitLab MR Review Skill

Автоматическое ревью Merge Request'ов в GitLab с комплексным анализом кода.

## Быстрый старт

1. **Настройка (один раз):**
   ```bash
   claude-tools review-setup
   ```

   Команда запросит Personal Access Token с правом `api`.

2. **Использование:**
   ```
   /review-mr https://gitlab.com/your-project/-/merge_requests/123
   ```

## Что проверяет

- ✅ Python best practices (PEP 8, type hints, idioms, imports, structure)
- ✅ Kotlin idioms (null safety, coroutines, scope functions, data classes, imports)
- ✅ Go idioms (error handling, concurrency, goroutines, channels, naming)
- ✅ SQL queries (performance, N+1, indexes, schema design, ORM anti-patterns)
- ✅ Security vulnerabilities (SQL injection, XSS, secrets, OWASP Top 10)
- ✅ Test quality (coverage, edge cases, assertions)
- ✅ Code quality (complexity, duplication, naming)
- ✅ Performance issues (N+1 queries, inefficient loops)
- ✅ Airflow specific (DAG parsing, lazy imports, idempotency)
- ✅ Project structure (no utils/, constants organization, naming conventions)

## Workflow

1. Запускаете `/review-mr <url>`
2. Claude анализирует diff по всем политикам
3. Показывает результаты в CLI с severity: 🔴 CRITICAL, 🟡 MAJOR, 🔵 MINOR, ⚪ NIT
4. Спрашивает через форму: опубликовать всё, выбрать конкретные, или отменить
5. Публикует выбранные комментарии в GitLab MR

## Структура

```
.claude/skills/review-mr/
├── SKILL.md                      # Инструкции для Claude
├── README.md                     # Это файл
└── policies/                     # Политики ревью
    ├── python-review-policy.md   # Python best practices + Airflow
    ├── kotlin-review-policy.md   # Kotlin idioms + coroutines
    ├── go-review-policy.md       # Go idioms + concurrency
    ├── sql-review-policy.md      # SQL queries + schema + ORM
    ├── security-policy.md        # Security (OWASP Top 10, CWE)
    ├── tests-policy.md           # Test quality (pytest)
    └── general-policy.md         # General code quality
```

## Настройка токена

GitLab аутентификация настраивается через glab CLI.

**Необходимое право:** `api`

Создать токен: https://gitlab.ru/-/user_settings/personal_access_tokens

## Кастомизация

Политики можно редактировать под нужды команды:

- Добавить специфичные правила
- Изменить severity
- Отключить неприменимые правила
- Добавить примеры из вашего кодбейза

## Источники политик

Основаны на индустриальных стандартах:

- Python: PEP 8, pylint/flake8
- Kotlin: Kotlin Coding Conventions, idioms
- Go: Google Go Style Guide, Effective Go
- SQL: Database optimization best practices
- Security: OWASP Top 10, CWE
- Tests: pytest best practices
- General: Clean Code, Refactoring patterns
- Airflow: Official best practices

## Требования

- glab CLI установлен (`brew install glab`) — либо доступна обёртка `dp glab` (тот же синтаксис команд), которую можно использовать вместо прямого вызова `glab`
- GitLab Personal Access Token с правом `api`
- Доступ к GitLab по сети

## Troubleshooting

**"glab not found":**
```bash
brew install glab
```

**"Token not found":**
```bash
claude-tools review-setup
```

**Много ложных срабатываний:**
Отредактируйте политики в `./policies/`, измените severity или уберите правила.
