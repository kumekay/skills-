---
name: using-tea-for-gitea
description: Use `tea` CLI for all interactions with Gitea instances, including repositories, issues, pull requests, actions, releases, labels, milestones, and branches. Use when the user mentions a Gitea host, shares a Gitea URL, asks about Gitea repos, or wants to create/manage issues, PRs, or actions on Gitea. Also use when the current git remote points to a Gitea host. Prefer `tea` over browser tools or generic web fetching for any Gitea data.
---

# Using `tea` For Gitea

## Rule

When interacting with a Gitea instance, use the `tea` CLI. This includes issues, pull requests, releases, actions, branches, labels, milestones, webhooks, and any other Gitea entities.

A single Gitea instance may be reachable via multiple hostnames and may use a non-standard SSH port (e.g. `ssh://git@host:PORT/owner/repo.git`). Treat these as the same instance.

- Do not use browser-style fetching or generic web tools for a Gitea host when `tea` can provide the data.
- Do not guess API paths when `tea help <command>` can answer directly.
- For REST API access not covered by dedicated commands, use `tea api`.

## Quick Reference

```bash
# Issues
tea issues list                       # list issues
tea issues create --title "..." --body "..."  # create issue
tea issues details <number>           # view issue details

# Pull Requests
tea pulls list                        # list PRs
tea pulls create --title "..." --body "..."   # create PR
tea pulls details <number>            # view PR details
tea pulls checkout <number>           # checkout PR locally
tea pulls merge <number>              # merge PR

# Actions
tea actions list                      # list action runs
tea actions details <run-id>          # view run details

# Other entities
tea repos list                        # list repos
tea releases list                     # list releases
tea branches list                     # list branches
tea labels list                       # list labels
tea milestones list                   # list milestones
tea comment <number> --body "..."     # comment on issue/PR

# Raw API access
tea api <endpoint>                    # authenticated API request
tea api repos/owner/repo              # example: get repo info
```

## Long-form bodies (issues, PRs, comments)

Do NOT pass multi-paragraph markdown into `--body "..."` on the command line.
Shell escaping mangles backticks, quotes, and code fences — the rendered issue
ends up with literal `\` before every backtick and `"`. Write the body to a
file and read it through `tea api`'s `-F field=@file` form, which sends file
contents verbatim:

```bash
# Create an issue with a long markdown body
tea api -X POST repos/{owner}/{repo}/issues \
  -f title="..." \
  -F body=@/tmp/issue-body.md

# Create a PR the same way
tea api -X POST repos/{owner}/{repo}/pulls \
  -f title="..." \
  -F body=@/tmp/pr-body.md \
  -f head="feature-branch" \
  -f base="main"

# Add a long comment
tea api -X POST repos/{owner}/{repo}/issues/{n}/comments \
  -F body=@/tmp/comment.md
```

`-f` is for plain string fields; `-F` accepts `@file` to read from disk
(`@-` for stdin). For one-line titles `-f` is fine; for markdown bodies
always use `-F ...=@file`.

## Editing existing issues and PRs

`tea` doesn't have a dedicated `edit` subcommand for most fields. Use
`tea api -X PATCH` against the resource:

```bash
# Edit an issue body or title
tea api -X PATCH repos/{owner}/{repo}/issues/{n} -F body=@/tmp/new-body.md
tea api -X PATCH repos/{owner}/{repo}/issues/{n} -f title="new title"

# Edit a PR (PRs share the issues edit endpoint for body/title;
# branch/base/state edits go through pulls/{n})
tea api -X PATCH repos/{owner}/{repo}/issues/{n} -F body=@/tmp/new-body.md
tea api -X PATCH repos/{owner}/{repo}/pulls/{n} -f state="closed"

# Close or reopen
tea api -X PATCH repos/{owner}/{repo}/issues/{n} -f state="closed"
tea api -X PATCH repos/{owner}/{repo}/issues/{n} -f state="open"
```

After editing, re-fetch with `tea api repos/{owner}/{repo}/issues/{n}` to
verify the body rendered cleanly with no stray escape characters.

## Context Detection

`tea` automatically detects the repository from the current directory's git remote. When the user is working in a repo hosted on a configured Gitea instance, `tea` commands work without specifying the repo explicitly.

If the user is not in a repo directory, or needs to target a different repo, use `--repo owner/repo` or `--login <login-name>` to select a configured Gitea login.

## URL Handling

If the user gives a Gitea URL, parse the owner and repo from the URL path, extract the resource type and ID, and use the appropriate `tea` command.

Examples (replace `<host>` with the actual Gitea hostname):

- Issue URL: `https://<host>/owner/repo/issues/5`
  Use: `tea issues details 5 --repo owner/repo`

- PR URL: `https://<host>/owner/repo/pulls/3`
  Use: `tea pulls details 3 --repo owner/repo`

- Repo URL: `https://<host>/owner/repo`
  Use: `tea repos details --repo owner/repo` or `tea api repos/owner/repo`

## Workflow

1. Check if the requested data lives on a Gitea host (via URL, git remote, or user context).
2. If yes, use `tea`.
3. If a dedicated `tea` subcommand exists, prefer it.
4. If not, use `tea api` for direct API access.
5. If access fails, report the error clearly instead of switching tools.

## Failure Handling

- If `tea` is not authenticated, say so and suggest `tea login add`.
- If a command returns an error, check `tea help <command>` for correct syntax.
- If the API returns 404, verify the owner/repo and resource ID.
- If the API returns 403, report a permissions problem rather than falling back to another tool.

## Common Mistakes

- Do not use `gh` (GitHub CLI) for Gitea repositories — use `tea`.
- Do not use generic web fetch tools for Gitea data when `tea` can answer.
- Do not switch away from `tea` just because a dedicated subcommand seems missing; try `tea api` first.
- Do not pass long markdown bodies via `--body "..."` — shell escaping mangles backticks, quotes, and code fences. Write to a file and use `tea api ... -F body=@file.md` instead.
- Do not assume `tea` has an `edit` subcommand for an issue/PR field — use `tea api -X PATCH` against the resource.
