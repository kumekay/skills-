# Skills Repository

This repository contains agent skills installable via the `skills` CLI.

## Structure

Each skill lives in its own directory with a `SKILL.md` file that contains the skill's metadata (frontmatter) and instructions.

## Adding a New Skill

1. Create a new directory named after the skill (kebab-case)
2. Add a `SKILL.md` with frontmatter (`name`, `description`) and instructions
3. Update the table in `README.md`

## Conventions

- Skill directories use kebab-case naming
- Each `SKILL.md` must have YAML frontmatter with `name` and `description`
- Keep skill instructions focused and actionable
