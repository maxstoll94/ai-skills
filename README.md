# AI Skills

This repo contains Codex skills that can be installed into `~/.codex/skills`.

## Included skills

- `wbso-skill`: AI-led WBSO dry run and apply workflow using `gh` and `linear`, with defaults for the common repos, authenticated assignees, BAC/OPS Linear scope, and roughly 20 hours per week

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

## Defaults

Unless you override them in the prompt, `wbso-skill` assumes:

- repositories: `MatchWornShirt/monorepo` and `360ERP/mws-ftg`
- GitHub assignee: the logged-in `gh` user
- Linear assignee: the logged-in `linear` user
- Linear team search scope: `BAC` and `OPS`

## Example prompt

```text
Use $wbso-skill for 2026-01-01 to 2026-03-14, holidays on 2026-02-05 and 2026-02-06.
```
