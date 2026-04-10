# Feed — SNS Content Generator & Publisher Plugin

## Overview
Claude Code plugin that generates platform-optimized SNS content via interview-driven workflow.

## Architecture
- **Plugin (this repo)**: read-only templates — skills, rules, platforms, styles, flows
- **User data (`~/.config/feed/`)**: config, posts, analytics — never committed to repo

## Skills
- `/feed` — Main. Interview → generate → save → publish
- `/feed-publish` — Publish unpublished posts
- `/feed-report` — Period-based analytics report
- `/feed-recommend` — Topic recommendation

## Directory Guide
- `skills/` — Skill definitions (SKILL.md + supporting files)
- `rules/` — Writing rules (AI prevention, scoring, language-specific)
- `platforms/` — Platform profiles (characteristics, algorithm, API info)
- `styles/` — Content style templates (structure, examples, scoring)
- `flows/` — Workflow definitions (save format, publish logic)
- `docs/design.md` — Full design document

## How Skills Load Files
SKILL.md is a lightweight router. It reads `reference.md` to determine which files to load:
- Always: `rules/common.md`, `rules/scoring.md`
- Per language: `rules/{lang}.md`
- Per platform: `platforms/{platform}.md`
- Per style: `styles/{style}.md`
- On save: `flows/save.md`
- On publish: `flows/publish.md`

## Local Testing
```bash
claude --plugin-dir ./feed
```

## Key Principle
Generated content must feel human-written. If it reads like AI output, it's a failure.
