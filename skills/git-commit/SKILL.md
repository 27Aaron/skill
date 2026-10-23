---
name: git-commit
description: Write, review, or fix Git commit messages (including commitlint errors), and create or amend commits when explicitly asked. Not for PR titles or descriptions.
metadata:
  author: Aaron
  version: "2.0.0"
---

# Git Commit

Follow user instructions first, then repo rules (`AGENTS.md`, `CONTRIBUTING`, commitlint config), then the style of recent `git log`. Otherwise use Conventional Commits.

## Scope

- **Draft**: reply with only the message as plain text — no preamble, quotes, or code fence. Don't touch the repo.
- **Review**: list the problems and give a corrected message.
- **Commit / amend**: only when explicitly asked. Amend only `HEAD`.
- Never push, change Git config, or use `--no-verify` unless asked.

## Format

```text
<type>(<scope>)!: <description>

<body>

<footer>
```

**Type**: `feat`, `fix`, `perf`, `refactor` (no behavior change), `docs`, `test`, `build` (build system, dependencies), `ci`, `style` (formatting only), `revert`, `chore` (last resort). Choose by effect, not file paths: tests and docs that accompany a feature stay in `feat`.

**Subject**:

- Whole line ≤ 50 characters if possible, never over 72, unless the repo sets its own limit.
- Imperative, lowercase description, no trailing period: `fix(auth): reject expired refresh tokens`, not `fix: Updated auth.ts.`
- Add a scope only when the module is obvious from the diff or repo convention.
- Match the repo's language; default to English.

**Body**: only when the subject can't explain *why*. Blank line after the subject, wrap at 72, don't list files.

**Footer**:

- Breaking change: `!` before the colon and/or `BREAKING CHANGE: <what breaks, how to migrate>`.
- Reference issues only when the number is known — never invent one. Add trailers like `Co-authored-by:` only when asked or required.
- Revert: `revert: <original subject>` with body `This reverts commit <sha>.`

```text
fix(sync): retry uploads after token refresh

Uploads started just before token expiry failed with 401 and were
dropped silently. Refresh the token once and replay the request.
```

```text
refactor(config)!: drop YAML config support

BREAKING CHANGE: config.yaml is no longer read; use config.toml.
```

## Committing

1. If the user gave a diff or summary, use it. Otherwise describe the staged diff (`git diff --staged`) only; if nothing is staged, say so. Don't invent motivation the diff doesn't show.
2. When asked to commit, check `git status --short` and `git diff`. Stage only the requested changes with `git add <path>`; never stage secrets, `.env`, or unrelated files. Don't unstage what the user already staged without asking.
3. Suggest splitting unrelated changes, but don't split without approval.
4. Commit via stdin:
   ```bash
   git commit --file=- <<'EOF'
   feat(api): add cursor pagination to /orders
   EOF
   ```
5. If a hook fails, fix in-scope issues and retry; stop after a repeated failure.
6. Report `git log -1 --format='%h %s'` and anything left uncommitted.

**Amend**: `git commit --amend --only --file=-` changes the message only; to add changes, stage them and run `git commit --amend --file=-`, describing the whole commit. Warn first if `HEAD` is already pushed.

**commitlint errors**: fix only what the failing rule reports, keeping the meaning. Don't edit the config or disable rules.
