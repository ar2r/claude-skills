# ar2r-skills Plugin

Workflow collection for professional bug fixing and GitLab merge request descriptions.

## Included Skills

### `create-mr`

Creates a Merge Request in GitLab from the current branch: asks only whether the MR should be draft, then auto-generates the title and structured description, pushes the branch, and publishes the MR via `glab`. If an MR for the current branch already exists, it updates that MR description instead of creating a new one.

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
/plugin marketplace add ar2r/agent-skills
/plugin install ar2r-skills@ar2r-skills
```

### Option 2: Manual Installation

Copy the plugin directory to your Claude plugins folder:

```bash
cp -r plugins/ar2r-skills ~/.claude/plugins/
```

### Qwen / Nessy CLI

Install from the repository as a native, updatable Qwen extension:

```bash
qwen extensions install https://github.com/ar2r/agent-skills/ --ref=master
qwen extensions update ar2r-skills
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
- `create-mr` asks only for draft mode when creating a new MR, auto-generates the title and description, and updates the existing MR description if the branch is already on review.
- `describe-mr` generates a review-ready Markdown description for an existing MR and optionally publishes it to GitLab.
- For MR workflows, `glab` should be installed and authenticated against the target GitLab instance.

## License

MIT
