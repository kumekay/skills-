# Agent Skills(-)


My humble collection of [agent skills](https://agentskills.io) for AI coding assistants, installable via the [`skills` CLI](https://github.com/vercel-labs/skills).

> [!TIP]
> **Browse the full catalog at [skills.kumekay.com](http://skills.kumekay.com/)** — a searchable index of these skills plus skills from other repos, with one-click install commands.

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
| [using-cloner](using-cloner/SKILL.md) | Use [`clone`](https://github.com/kumekay/cloner) instead of `git clone` to clone repos or get the local directory of an existing checkout. |
| [using-tea-for-gitea](using-tea-for-gitea/SKILL.md) | Use the `tea` CLI for all Gitea interactions — issues, PRs, actions, releases, and raw API access. |
