# Agent Skills(-)


My humble collection of [agent skills](https://agentskills.io) for AI coding assistants, installable via the [`skills` CLI](https://github.com/vercel-labs/skills).

## Installation

Requires Node.js 18+. Install all skills globally:

```bash
npx skills add git@github.com:kumekay/skills-.git -g
```

Or install a specific skill:

```bash
npx skills add git@github.com:kumekay/skills-.git --skill undercover -g
```

To install into the current project instead, drop the `-g` flag.

## Available Skills

| Skill | Description |
|-------|-------------|
| [undercover](undercover/SKILL.md) | Operate without revealing AI identity. Strips all AI attribution from commits, PRs, code comments, and public-facing text. |
| [using-tea-for-gitea](using-tea-for-gitea/SKILL.md) | Use the `tea` CLI for all Gitea interactions — issues, PRs, actions, releases, and raw API access. |
