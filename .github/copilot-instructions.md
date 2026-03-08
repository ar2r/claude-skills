# Copilot Instructions

## What this repo is

A collection of **Claude Code skills and plugins**. Skills are instruction files (SKILL.md) that teach Claude Code how to perform structured workflows. Plugins group related skills and are distributed via the Claude Code plugin marketplace.

## Repository structure

```
.claude-plugin/marketplace.json          # Top-level marketplace registry
plugins/<plugin-name>/
  README.md
  skills/<skill-name>/
    SKILL.md                             # The skill itself (YAML frontmatter + Markdown)
    evals/evals.json                     # Evaluation test cases for the skill
```

## Key conventions

### SKILL.md frontmatter

Every SKILL.md starts with a YAML frontmatter block that controls when Claude Code auto-triggers the skill:

```yaml
---
name: skill-name
description: |
  Prose description used by Claude to decide when to invoke this skill.
  Include explicit trigger phrases (in both English and Russian if needed).
  Include "Examples:" section with user → action pairs.
---
```

The `description` field is the triggering signal — write it to match the natural-language phrases users actually type, including non-English triggers where applicable.

### Plugin metadata files

- `marketplace.json` — top-level Claude Code marketplace registry; lists plugins, source paths, metadata, and component paths. Update when adding or renaming plugins.
- Plugin-local `.claude-plugin/plugin.json` is optional. This repo keeps plugin metadata in the marketplace entry instead, with `"strict": false`, to avoid duplicate manifests.

### Evals

`evals/evals.json` contains an array of `{id, name, prompt, expected_output, files}` objects. Add an eval for every distinct trigger scenario the skill handles. `expected_output` is a prose description of what the skill should do — not a literal string match.

### Language conventions

- Skill/plugin names and file names: English, kebab-case
- `describe-mr` skill output: **Russian** (including all section headers). Technical terms (API, endpoint, middleware) stay in English.
- `fix-bug` skill output: English, with Russian phrases accepted as input triggers.

### Plugin `strict: false`

The `dev-pro` plugin is marked `"strict": false` in `marketplace.json`. This makes the marketplace entry the full plugin definition, so the plugin does not need its own `.claude-plugin/plugin.json`.

## Installation command (for reference)

```bash
/plugin marketplace add ar2r/agent-skills
/plugin install dev-pro@ar2r-agent-skills
/plugin marketplace update ar2r-agent-skills
```

## External dependencies

- `glab` — required by `describe-mr` to publish descriptions to GitLab (`glab mr update`). Must be installed and authenticated.
