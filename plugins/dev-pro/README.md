# dev-pro Plugin

Workflow collection for professional bug fixing and GitLab merge request descriptions.

## Included Skills

### `create-mr`

Creates a Merge Request in GitLab from the current branch: pushes the branch, generates a structured description, and publishes the MR via `glab`.

Typical triggers:

- "Создай MR", "открой merge request", "запушь ветку и создай MR"
- "Сделай PR/MR в GitLab", "залей ветку на ревью"
- "create MR", "open pull request", "push and create MR"

### `fix-bug`

Structured 7-phase workflow for bug fixing:

1. **Information Gathering** - Collect bug details systematically
2. **Bug Categorization** - Classify by type (logic, race condition, memory, etc.)
3. **Investigation** - Search code, analyze git history, reproduce the issue
4. **Fix Implementation** - Minimal safe fixes with side-effect analysis
5. **Test Coverage** - Add regression tests that would have caught the bug
6. **Validation Checklist** - Code quality, testing, impact assessment
7. **Rollback Plan** - Prepare for worst-case scenarios

Typical triggers:

- "There's a bug in my code"
- "Fix this error"
- "The tests are failing"
- "This crashes when I click X"
- "почини баг", "исправь ошибку", "debug", "failing test", "not working"

### `describe-mr`

Generates a structured Merge Request description in Russian for an **existing** MR from GitLab metadata, commits, and diffs. Can publish the final text with `glab mr update`. To create a new MR, use `create-mr`.

Typical triggers:

- `/describe-mr https://gitlab.example.com/team/app/-/merge_requests/123`
- "Сгенерируй описание PR для ревью"
- "Нужен текст для MR"
- "Обнови описание merge request в GitLab"

## Installation

### Option 1: Via Claude Code CLI (Recommended)

```bash
/plugins add https://github.com/ar2r/claude-skills
```

### Option 2: Manual Installation

Copy the plugin directory to your Claude plugins folder:

```bash
cp -r plugins/dev-pro ~/.claude/plugins/
```

## Usage

Examples:

```bash
Fix the null pointer exception in UserService
Debug why the payment is failing
This test is failing, can you fix it?
Создай MR из текущей ветки
/describe-mr https://gitlab.example.com/team/app/-/merge_requests/123
Сгенерируй описание PR для ревью
```

## Notes

- `fix-bug` focuses on diagnosis, implementation, regression coverage, and rollback planning.
- `create-mr` pushes the current branch, generates a description, and publishes the MR to GitLab.
- `describe-mr` generates a review-ready Markdown description for an existing MR and optionally publishes it to GitLab.
- For MR workflows, `glab` should be installed and authenticated against the target GitLab instance.

## License

MIT
