# Claude Skills

Плагины и skills для Claude Code.

## Установка

```
/plugins add https://github.com/ar2r/claude-skills
```

## Обновление

```
/plugins update dev-pro
```

Если плагин уже был установлен под старым именем `fix-bug`, переустанови его из репозитория после обновления.

## Доступные плагины

| Плагин | Описание |
|--------|----------|
| **dev-pro** | Набор workflows для исправления багов и подготовки описаний Merge Request в GitLab |

## Skills в `dev-pro`

| Skill | Описание |
|-------|----------|
| **fix-bug** | Профессиональный workflow для расследования багов, безопасного фикса и регрессионного покрытия |
| **create-mr** | Создаёт MR в GitLab из текущей ветки: пуш, генерация описания и публикация через `glab` |
| **describe-mr** | Генерация структурированного описания для существующего MR/PR на русском языке с опцией публикации через `glab` |

## Триггеры

### `fix-bug`

Автоматически активируется на фразы:

```
"fix this bug", "debug", "failing test", "not working", "crashes when"
"почини баг", "исправь ошибку", "найди причину"
```

### `create-mr`

Активируется, когда пользователь:

```
просит создать MR/PR из текущей ветки
"создай MR", "открой merge request", "залей на ревью"
"запушь ветку и создай MR", "сделай PR в GitLab"
```

### `describe-mr`

Активируется, когда пользователь:

```
передаёт ссылку на GitLab MR
просит написать или обновить описание существующего PR/MR
просит подготовить текст изменений для ревью
упоминает "changelog ветки" или "текст для MR/PR"
```

## Структура

```
claude-skills/
├── .claude-plugin/marketplace.json
└── plugins/dev-pro/
    ├── README.md
    ├── .claude-plugin/plugin.json
    └── skills/
        ├── fix-bug/
        │   └── SKILL.md
        ├── create-mr/
        │   └── SKILL.md
        └── describe-mr/
            └── SKILL.md
```

MIT License · Artur Khasanov
