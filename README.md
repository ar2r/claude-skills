# Claude Skills

Плагины для Claude Code.

## Установка

```
/plugins add https://github.com/ar2r/claude-skills
```

## Обновление

```
/plugins update fix-bug-pro
```

## Доступные плагины

| Плагин | Описание |
|--------|----------|
| **fix-bug-pro** | Профессиональный workflow для исправления багов: investigation → fix → regression tests → rollback plan |

## Триггеры

Автоматически активируется на фразы:

```
"fix this bug", "debug", "failing test", "not working", "crashes when"
"почини баг", "исправь ошибку", "найди причину"
```

## Структура

```
claude-skills/
├── .claude-plugin/marketplace.json
└── plugins/fix-bug-pro/
    ├── .claude-plugin/plugin.json
    ├── skills/fix-bug-pro/SKILL.md
    └── evals/evals.json
```

MIT License · Artur Khasanov