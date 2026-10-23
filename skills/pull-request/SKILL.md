---
name: pull-request
description: Write or review PR branch names, titles, and descriptions, and create, update, or merge GitHub PRs when asked. For commit messages alone, use git-commit.
metadata:
  author: Aaron
  version: "2.0.0"
---

# Pull Request

Follow user instructions first, then repo rules (`AGENTS.md`, `CONTRIBUTING`, PR template, PR title checks), then the style of recent PRs. Otherwise use the defaults below.

## Scope

- **Draft** (name, title, body): return the text only. Don't push or touch the remote.
- **Create**: branch, commit, push, and open the PR as needed — no need to confirm each step.
- **Update**: change only the requested fields.
- Merge, close, delete branches, or force-push only when asked. Keep the user's unrelated and uncommitted changes intact.

## Changes

- Base: the user's choice, else the repo's default branch. In a fork, `origin` may not be the upstream.
- Describe the whole branch, not the last commit: `git diff --stat <base>...HEAD` and `git diff <base>...HEAD`.
- Uncommitted changes aren't part of the PR unless the user asks to include them.

## Branch name

`<type>/<short-description>`, lowercase kebab-case: `feat/oauth-login`, `fix/123-expired-token`, `docs/api-guide`.

- Types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `ci`, `chore`, `hotfix`, `release`.
- Keep it short; include an issue number only if one exists.
- Don't rename an existing PR branch — it closes the PR.

## Title

Same rules as a Conventional Commits subject: `feat(auth): add refresh-token rotation`.

- Imperative, lowercase description, no trailing period, ≤ 72 characters.
- Describe the main outcome; supporting tests and docs don't change the type.
- Squash merges usually reuse the title as the commit subject, so mark breaking changes with `!`.
- Unrelated goals in one PR: suggest splitting.

## Description

Use the repo's PR template if present and fill every required section. Otherwise:

```markdown
## Overview

<Problem and resulting behavior in 1–3 sentences.>

## Changes

- <Notable change and why.>

## Validation

- <Command or scenario> — <result>
```

- Write for a reviewer who hasn't seen the conversation; don't list every file.
- Add sections only when they matter: breaking changes and migration, risks, screenshots for UI, benchmarks for performance claims.
- Validation must be honest: say what you ran, what CI reports, and what wasn't tested. Never claim tests passed without evidence.
- `Closes #123` only when the PR fully fixes a known issue and targets the default branch; otherwise `Refs #123`. Never invent issue numbers.
- Keep the title and body in sync if the scope changes.

## GitHub workflow

1. Check for an existing PR first: `gh pr list --head <branch> --state all`. Don't reuse a closed or merged one.
2. Before pushing, review every commit that will be pushed; run relevant lint, tests, or build, and `git diff --check <base>...HEAD`.
3. Push only the PR branch: `git push -u origin <branch>`.
4. Create with explicit arguments and a body file — no `--fill`, no inline body:
   ```bash
   gh pr create --repo <owner/repo> --base <base> --head <branch> \
     --title "<title>" --body-file <file>
   ```
5. To update, read the current body first (`gh pr view <n> --json title,body`), then `gh pr edit <n> --title … --body-file …`, preserving others' content.
6. Use `--draft` when work is unfinished or needs discussion.
7. Verify with `gh pr view <n>` and report the URL.

**Reviews**: fix in-scope feedback, push, and reply. List out-of-scope suggestions as follow-ups instead of expanding the PR.

**Merge** (only when asked): confirm required checks and approvals pass on the latest head (`gh pr checks <n>`), then merge with the repo's method. Auto-merge or a merge queue isn't merged yet. Delete the branch only after the merge is confirmed.

If a command times out, check whether it took effect before retrying, so you don't create a duplicate PR.
