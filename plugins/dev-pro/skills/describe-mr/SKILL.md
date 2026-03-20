---
name: describe-mr
description: >
  Генерирует описание Merge Request на русском языке и публикует его в GitLab
  через glab. Используй этот скилл когда пользователь передаёт ссылку на MR
  (например /describe-mr https://gitlab.../merge_requests/123), или просит
  написать, обновить, сгенерировать описание PR/MR для существующего мерж-реквеста.
  Для создания нового MR — используй скилл create-mr.
---

# MR Description

Скилл генерирует описание для уже существующего MR и публикует его через `glab`.

---

## Шаг 1 — Определи режим

**Режим A — передан URL:**
```
/describe-mr https://gitlab.company.com/team/repo/-/merge_requests/42
```
Извлеки из URL: **host**, **namespace/repo**, **mr_id**.

**Режим Б — без аргументов (текущая ветка):**
```bash
git branch --show-current          # source_branch
git remote get-url origin          # → host, namespace/repo
glab mr list --source-branch <source_branch> --repo <host>/<namespace>/<repo> --state opened
```
Если MR найден — сохрани **mr_id**. Если не найден — предупреди и продолжи без mr_id (описание для копирования).

Если не в git-репозитории или glab не аутентифицирован — останови с сообщением об ошибке.

---

## Шаг 2 — Получи ветки MR

```bash
glab mr view <mr_id> --repo <host>/<namespace>/<repo>
```

Извлеки **source_branch** и **target_branch**.

В Режиме Б без mr_id — определи target_branch:
```bash
git remote show origin | grep "HEAD branch" | awk '{print $NF}'
```

---

## Шаг 3 — Собери diff

```bash
git log origin/<target_branch>..origin/<source_branch> --oneline --no-merges
git diff --stat origin/<target_branch>...origin/<source_branch>
git diff origin/<target_branch>...origin/<source_branch>
```

Если diff >500 строк — читай точечно по ключевым файлам:
```bash
git diff origin/<target_branch>...origin/<source_branch> -- <path/to/file>
```

Если локальных веток нет — опирайся на данные из `glab mr view`.

---

## Шаг 4 — Сгенерируй описание

Проанализируй изменения: что сделано, зачем, как реализовано, что может сломаться, как проверить.

Сгенерируй по шаблону. **Пустые секции опускай.**

```markdown
## 📋 Описание
<2–4 предложения: ЧТО сделано и ЗАЧЕМ. Контекст, а не пересказ кода.>

## 🔧 Тип изменений
- [ ] ✨ feat  - [ ] 🐛 fix  - [ ] ♻️ refactor  - [ ] 📦 chore  - [ ] 📝 docs  - [ ] ✅ test  - [ ] ⚡ perf

## 📝 Что изменено
- **<Модуль>**: <что и зачем>

## 💡 Технические детали
<Только нетривиальные решения. Если очевидно — опустить.>

## ⚠️ Breaking Changes
<Если нет — опустить.>

## 🗄️ Миграции / Конфигурация
<Если нет — опустить.>

## ✅ Как проверить
1. <шаг>
2. Ожидаемый результат: <что должно произойти>

## 🧪 Тесты
<Что покрывают тесты. Если нет — причина. Если неприменимо — опустить.>

## 🔗 Связанные задачи
<Ссылки на тикеты. Если нет — опустить.>
```

---

## Шаг 5 — Интерактивный цикл

Выведи описание и спроси:

> **Что делаем?**
> 1. 🚀 Опубликовать в MR !<mr_id>
> 2. ✅ Готово (оставить себе)
> 3. ✏️ Доработать — напиши правки

- **Опубликовать:**
  ```bash
  cat > /tmp/mr_desc.md << 'EOF'
  <описание>
  EOF
  glab mr update <mr_id> --repo <host>/<namespace>/<repo> --description "$(cat /tmp/mr_desc.md)"
  ```
  Сообщи: `✅ Описание опубликовано в MR !<mr_id>`

- **Доработать:** прими правки, обнови описание, повтори цикл.

---

## Правила написания

- Русский язык, технические термины (API, endpoint, middleware) — как есть
- «Зачем» важнее «что» — контекст ценнее перечисления файлов
- Группируй по логическому модулю, не по именам файлов
- Отмечай чекбоксы: `[x]`
- Не пиши «изменён файл `src/utils.ts`» — это видно в диффе
- Не оставляй секции-заглушки
