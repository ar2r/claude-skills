# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of **Claude Code skills and plugins**. Skills are instruction files (`SKILL.md`) that teach AI agents how to perform structured workflows. Plugins group related skills and are distributed via the Claude Code plugin marketplace and as native Qwen/Nessy extensions.

## Repository structure

```
.claude-plugin/marketplace.json          # Claude Code marketplace registry
qwen-extension.json                      # Qwen/Nessy extension manifest
nessy-extension.json                     # Nessy extension manifest
skills/<skill-name>/
  SKILL.md                               # The skill itself (YAML frontmatter + Markdown)
  references/                            # Optional reference docs read by the skill
  evals/evals.json                       # Evaluation test cases for the skill
```

## Adding or editing a skill

1. Create or edit `skills/<skill-name>/SKILL.md`.
2. The YAML frontmatter `description` field is the triggering signal — write it to match natural-language phrases users actually type (include Russian phrases where applicable):
   ```yaml
   ---
   name: skill-name
   description: |
     Prose description...
     Examples:
     - user: "..." → action
   ---
   ```
3. Add evals in `evals/evals.json` — an array of `{id, name, prompt, expected_output, files}` objects, one per distinct trigger scenario.
4. **No need to update `marketplace.json`** — the plugin uses `"strict": false` and auto-discovers skills from `./skills/`.

## Language conventions

| Artifact | Language |
|---|---|
| Skill/plugin names and file names | English, kebab-case |
| `describe-mr` output (all section headers) | Russian |
| `fix-bug` output | English (Russian phrases accepted as input) |
| `create-mr` output | Russian |
| Technical terms (API, endpoint, middleware) | English in all skills |

## Plugin metadata

- `marketplace.json` — top-level Claude Code marketplace registry; lists plugins, source paths, and component paths.
- `"strict": false` means the marketplace entry is the full plugin definition; no plugin-local `.claude-plugin/plugin.json` needed.
- Both `qwen-extension.json` files (root and `plugins/ar2r-skills/`) declare the native Qwen/Nessy extension. Update `version` in both when bumping the plugin version.

## External dependencies

- `glab` — required by `create-mr` and `describe-mr` to interact with GitLab (`glab mr create`, `glab mr update`, `glab mr list`, `glab mr view`). Must be installed and authenticated (`glab auth login`).

## Installation (for reference)

**Claude Code:**
```
/plugin marketplace add ar2r/agent-skills
/plugin install ar2r-skills@ar2r-skills
/plugin marketplace update ar2r-skills
```

**Qwen / Nessy CLI:**
```
qwen extensions install https://github.com/ar2r/agent-skills --ref=master
```
