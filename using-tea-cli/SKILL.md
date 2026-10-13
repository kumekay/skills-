---
name: using-tea-cli
description: Use the `tea` CLI for all interactions with Gitea and Forgejo instances (including Codeberg), such as repositories, issues, pull requests, actions, releases, labels, milestones, and branches. Use when the user mentions a Gitea or Forgejo host, shares a URL from one, or wants to create or manage issues, PRs, or actions there. Also use when the current git remote points to a Gitea or Forgejo host. Prefer `tea` over browser tools or generic web fetching for this data.
---

# Using the `tea` CLI

## Rule

When interacting with a Gitea or Forgejo instance, use the `tea` CLI. This includes issues, pull requests, releases, actions, branches, labels, milestones, webhooks, and any other entities.

Forgejo is a fork of Gitea and keeps the Gitea API (`/api/v1`), so `tea` works with both. In this document, "instance" means a Gitea or a Forgejo server. Most commands behave the same on both. See [Forgejo differences](#forgejo-differences) for the exceptions.

A single instance may be reachable via multiple hostnames and may use a non-standard SSH port (e.g. `ssh://git@host:PORT/owner/repo.git`). Treat these as the same instance.

- Do not use browser-style fetching or generic web tools for an instance when `tea` can provide the data.
- Do not guess API paths when `tea help <command>` can answer directly.
- For REST API access not covered by dedicated commands, use `tea api`.

## Quick Reference

```bash
# Issues
tea issues list                                      # list issues
tea issues create --title "..." --description "..."  # create issue (note: --description, not --body)
tea issues details <number>                          # view issue details
tea issues edit <number> --description "..."         # edit issue title/description/labels/…
tea issues close <number>                            # close issue
tea issues reopen <number>                           # reopen issue

# Pull Requests
tea pulls list                                       # list PRs
tea pulls create --title "..." --description "..."   # create PR (note: --description, not --body)
tea pulls details <number>                           # view PR details
tea pulls checkout <number>                          # checkout PR locally
tea pulls merge <number>                             # merge PR
tea pulls close <number>                             # close PR
tea pulls reopen <number>                            # reopen PR

# Actions (on Forgejo, see "Forgejo differences")
tea actions runs list                 # list workflow runs
tea actions runs view <run-id>        # view run details and jobs
tea actions runs logs <run-id>        # view run logs

# Other entities
tea repos list                        # list repos
tea releases list                     # list releases
tea branches list                     # list branches
tea labels list                       # list labels
tea milestones list                   # list milestones
tea comment <number> "..."                           # comment on issue/PR (body is positional)

# Raw API access
tea api <endpoint>                    # authenticated API request
tea api repos/owner/repo              # example: get repo info
```

## Long-form bodies (issues, PRs, comments)

For multi-paragraph markdown bodies, write the content to a temp file first,
then pass it via `--description "$(cat /tmp/file.md)"`. This avoids inline
shell-quoting headaches with backticks, code fences, and nested quotes:

```bash
# Create an issue with a long markdown body
tea issues create --title "..." --description "$(cat /tmp/issue-body.md)"

# Create a PR the same way
tea pulls create --title "..." --description "$(cat /tmp/pr-body.md)"

# Edit an existing issue body
tea issues edit <n> --description "$(cat /tmp/new-body.md)"

# Comment (body is positional)
tea comment <n> "$(cat /tmp/comment.md)"
```

For cases not covered by a dedicated subcommand (e.g. creating a PR with
extra fields, or editing a PR body), fall back to `tea api` with
`-F body=@file` which sends file contents verbatim:

```bash
tea api -X POST repos/{owner}/{repo}/pulls \
  -f title="..." \
  -F body=@/tmp/pr-body.md \
  -f head="feature-branch" \
  -f base="main"

tea api -X POST repos/{owner}/{repo}/issues/{n}/comments \
  -F body=@/tmp/comment.md
```

`-f` is for plain string fields; `-F` accepts `@file` to read from disk
(`@-` for stdin).

## Editing existing issues and PRs

**Issues** have a dedicated `tea issues edit` subcommand — prefer it for
short edits:

```bash
tea issues edit <n> --title "new title"
tea issues edit <n> --description "short updated body"
tea issues edit <n> --add-labels bug,urgent --milestone v1.2
tea issues close <n>    # state changes go through close/reopen
tea issues reopen <n>
```

For **long markdown** edits, write to a temp file and use `$(cat ...)`:

```bash
tea issues edit <n> --description "$(cat /tmp/new-body.md)"
```

**Pull requests** do **not** have a `tea pulls edit` subcommand. Use
`tea api -X PATCH` for any PR field edits. Title/body of a PR are edited via
the issues endpoint (PRs and issues share it for those fields); branch, base,
and state live on the pulls endpoint:

```bash
# Edit PR title / body (via the issues endpoint — same resource under the hood)
tea api -X PATCH repos/{owner}/{repo}/issues/{n} -F body=@/tmp/new-body.md
tea api -X PATCH repos/{owner}/{repo}/issues/{n} -f title="new title"

# Edit PR branch / base / state
tea api -X PATCH repos/{owner}/{repo}/pulls/{n} -f state="closed"

# Close / reopen a PR can also go through the dedicated subcommands
tea pulls close <n>
tea pulls reopen <n>
```

After any edit, re-fetch with `tea api repos/{owner}/{repo}/issues/{n}` (or
`.../pulls/{n}`) to verify the body was stored as intended.

## Context Detection

`tea` automatically detects the repository from the current directory's git remote. When the user is working in a repo hosted on a configured instance, `tea` commands work without specifying the repo explicitly.

If the user is not in a repo directory, or needs to target a different repo, use `--repo owner/repo` or `--login <login-name>` to select a configured login. Run `tea logins list` to see the configured logins and their URLs.

## URL Handling

If the user gives a URL from an instance, parse the owner and repo from the URL path, extract the resource type and ID, and use the appropriate `tea` command.

Examples (replace `<host>` with the actual hostname):

- Issue URL: `https://<host>/owner/repo/issues/5`
  Use: `tea issues details 5 --repo owner/repo`

- PR URL: `https://<host>/owner/repo/pulls/3`
  Use: `tea pulls details 3 --repo owner/repo`

- Repo URL: `https://<host>/owner/repo`
  Use: `tea repos details --repo owner/repo` or `tea api repos/owner/repo`

## Workflow

1. Check if the requested data lives on a Gitea or Forgejo host (via URL, git remote, or user context).
2. If yes, use `tea`.
3. If a dedicated `tea` subcommand exists, prefer it.
4. If not, use `tea api` for direct API access.
5. If access fails, report the error clearly instead of switching tools.

## Forgejo differences

To check which server you talk to, run `tea api version`:

- Gitea returns a plain version, for example `{"version":"1.27.3"}`.
- Forgejo returns its own version with a `+gitea-` suffix, for example `{"version":"16.0.5+gitea-1.22.0"}`. The suffix is the Gitea API level that Forgejo keeps compatible with.

Forgejo's actions API does not match what `tea` expects. On Forgejo:

- `tea actions runs list` works, but the WORKFLOW, BRANCH, and STARTED columns are empty.
- `tea actions runs view` and `tea actions runs logs` fail with `json: cannot unmarshal array into Go value of type gitea.ActionsWorkflowJobsResponse`.

Use `tea api` for actions on Forgejo:

```bash
# List runs with workflow name, branch, status, and web URL
tea api repos/{owner}/{repo}/actions/tasks

# Runs with workflow file (workflow_id) and status
tea api repos/{owner}/{repo}/actions/runs

# Jobs of one run (each job has an `id`)
tea api repos/{owner}/{repo}/actions/runs/<run-id>/jobs

# Logs of one job (plain text)
tea api repos/{owner}/{repo}/actions/jobs/<job-id>/logs
```

## Failure Handling

- If `tea` is not authenticated, say so and suggest `tea login add`.
- If a command returns an error, check `tea help <command>` for correct syntax.
- If the API returns 404, verify the owner/repo and resource ID.
- If the API returns 403, report a permissions problem rather than falling back to another tool.

## Common Mistakes

- Do not use `gh` (GitHub CLI) for Gitea or Forgejo repositories — use `tea`.
- Do not use generic web fetch tools for Gitea or Forgejo data when `tea` can answer.
- Do not switch away from `tea` just because a dedicated subcommand seems missing; try `tea api` first.
- Do not use `--body` for issues, PRs, or comments — it does not exist on these subcommands and will error with `flag provided but not defined: -body`. Use `--description` (or `-d`) for `tea issues create`/`tea pulls create`/`tea issues edit`. `tea comment` takes the body as a positional argument: `tea comment <n> "body text"`.
- For long markdown bodies, write to a temp file and use `--description "$(cat /tmp/body.md)"`, or fall back to `tea api ... -F body=@file.md` for subcommands that lack `--description`.
- Do not assume `tea pulls edit` exists — it does not. Use `tea api -X PATCH repos/{owner}/{repo}/issues/{n}` for PR title/body and `.../pulls/{n}` for branch/base/state. `tea issues edit` does exist and should be preferred for issue edits.
