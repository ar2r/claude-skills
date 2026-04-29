# Agent Skills

Плагины и skills для AI агентов.

## Установка в Claude code

```
/plugin marketplace add ar2r/agent-skills
/plugin install ar2r-skills@ar2r-skills
# Обновление
/plugin marketplace update ar2r-skills
```

## Установка в Qwen CLI (Nessy CLI)

```
qwen extensions install https://github.com/ar2r/agent-skills/ --ref=master
# or
nessy extensions install https://github.com/ar2r/agent-skills/ --ref=master

# Обновление
qwen extensions update --all
# or
nessy extensions update --all
```

Репозиторий содержит нативный `qwen-extension.json`, поэтому Qwen/Nessy устанавливает его как обновляемый git extension, а не как сконвертированный Claude plugin.

Если ранее extension был установлен до появления `qwen-extension.json` и в `/extensions manage` показывается `not updatable`, переустановите его один раз:

```
qwen extensions uninstall dev-pro
qwen extensions uninstall ar2r-agent-skills
qwen extensions uninstall ar2r-skills
qwen extensions install https://github.com/ar2r/agent-skills/ --ref=master
```

Важно: для Qwen/Nessy используется URL с завершающим `/` и без `:ar2r-skills`. Это заставляет CLI установить репозиторий как native git extension по `qwen-extension.json`, а не как Claude marketplace plugin.

## Доступные плагины

| Плагин | Описание |
|--------|----------|
| **ar2r-skills** | Набор workflows для исправления багов и подготовки описаний Merge Request в GitLab |

## Skills в `ar2r-skills`

| Skill | Описание |
|-------|----------|
| **fix-bug** | Профессиональный workflow для расследования багов, безопасного фикса и регрессионного покрытия |
| **create-mr** | Создаёт MR в GitLab из текущей ветки: один вопрос про draft для нового MR, затем автогенерация заголовка и описания; если MR уже есть, обновляет его описание |
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
agent-skills/
├── qwen-extension.json
├── skills -> plugins/ar2r-skills/skills
├── .claude-plugin/marketplace.json
└── plugins/ar2r-skills/
    ├── qwen-extension.json
    ├── README.md
    └── skills/
        ├── fix-bug/
        │   └── SKILL.md
        ├── create-mr/
        │   └── SKILL.md
        └── describe-mr/
            └── SKILL.md
```

Формат репозитория описан в документации Claude Code: [plugin marketplace format](https://code.claude.com/docs/en/plugin-marketplaces) и [plugin structure reference](https://code.claude.com/docs/en/plugins-reference).

MIT License · Artur Khasanov
