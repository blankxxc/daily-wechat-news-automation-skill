# Daily WeChat News Automation Skill

Hermes Agent skill for a daily WeChat Official Account news-article workflow.

This repository contains:

- `SKILL.md` — main skill instructions
- `references/` — workflow references and publishing pitfalls
- `templates/` — reusable cron/review prompt templates

## Usage

Copy this folder into a Hermes skills directory, then load the skill as:

```text
skill_view(name='daily-wechat-news-automation')
```

## Notes

The skill describes workflow structure, editorial constraints, image sourcing rules, WeChat draft/publishing gates, and common API pitfalls. It does not include credentials.
