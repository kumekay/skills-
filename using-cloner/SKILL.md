---
name: using-cloner
description: Use the `clone` CLI instead of `git clone` whenever you need to clone a repo or get the local directory of an existing checkout.
---

# Using `cloner`

## Rule

For **every** repository clone, use `clone <url>` — never `git clone <url>` directly.

`clone` is a small wrapper that:

1. Parses the git URL (HTTPS, SSH, `owner/repo` shorthand, custom SSH ports, nested groups).
2. Picks a structured target directory under the configured workspace (default `~/p`).
3. Clones the repo there, or, if a `.git` directory already exists at that path, skips the clone.
4. `cd`s into the resulting directory so subsequent commands run in the checkout.
5. Optionally applies per-repo `user.name` / `user.email` / `user.signingKey` based on the user's `~/.config/cloner.toml`.

Using `git clone` directly bypasses the workspace layout, the existence check, and the per-repo git identity config. Don't do it.

## Usage

```bash
clone <git-url-or-shorthand>
```

### Examples

```bash
# GitHub shorthand (uses SSH)
clone owner/repo

# HTTPS
clone https://github.com/owner/repo.git

# SSH
clone git@github.com:owner/repo.git

# GitLab / nested groups
clone git@gitlab.com:group/subgroup/repo.git

# Any non-GitHub host (hostname is included in the target path)
clone https://codeberg.org/owner/repo.git

# Self-hosted with a custom SSH port
clone ssh://git@host.example.com:2222/owner/repo.git
```

After `clone` finishes, the working directory is the fresh (or existing) checkout, so you can immediately run `git status`, `ls`, build commands, etc.

## When `clone` is Unavailable

`clone` is a shell function installed via `eval "$(clone --init)"` in the user's shell rc. If the function is not present:

- Report that `clone` is not set up rather than silently falling back to `git clone`.
- Suggest the user install it (`uv tool install git+https://github.com/kumekay/cloner`) and add `eval "$(clone --init)"` to their shell rc.
- Only fall back to plain `git clone` if the user explicitly asks for it.

## Working Directory Behavior

The shell function captures the target directory and `cd`s to it, so after a successful `clone` call the current directory is the checkout. In an agent shell where each command runs in its own invocation but the working directory persists, running `clone owner/repo` as a single command is enough — subsequent commands will execute inside the cloned repo.

If you need the target path explicitly (e.g. to pass it to another tool), run `clone <url> && pwd` in one invocation.

## Common Mistakes

- Running `git clone <url>` directly instead of `clone <url>`. Use `clone`.
- Running `git clone <url> some/custom/path` to pick a location — let `clone` decide the location from the workspace + URL. If a custom layout is truly needed, ask the user.
- Re-cloning a repo that already exists. `clone` is idempotent; just call it again and it will `cd` to the existing checkout.
- Forgetting that non-GitHub hosts get a hostname prefix in the target path. This is intentional — don't "fix" it by passing a custom path.
- Falling back to `git clone` silently when `clone` errors. Report the error and let the user decide.
