# AI Skills

This repo contains Codex skills that can be installed into `~/.codex/skills`.

## Included skills

- `wbso-skill`: AI-led WBSO dry run and apply workflow using `gh` and `linear`

## Install

Install the skill with Codex's `skill-installer`:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo maxstoll94/ai-skills \
  --path wbso-skill
```

Restart Codex after installing so the new skill is picked up.

## Requirements

- `gh` authenticated to GitHub
- `linear` authenticated to the correct Linear workspace
- access to the relevant GitHub repositories and Linear teams

## Example prompt

```text
Use $wbso-skill for 2026-01-01 to 2026-03-14, repo MatchWornShirt/monorepo, assignee maxstoll94, Linear team BAC, expected 180 hours, holidays on 2026-02-05 and 2026-02-06.
```
