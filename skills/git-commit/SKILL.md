---
name: git-commit
description: Write, review, or fix Git commit messages (including commitlint errors), and create or amend commits when explicitly asked. Follows repo conventions first, Conventional Commits otherwise. Not for PR titles or descriptions.
metadata:
  author: Aaron
  version: "2.0.0"
---

# Git Commit

Convention priority: user instructions > repo rules (`AGENTS.md`, `CONTRIBUTING`, commitlint config) > style of recent `git log` > [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/). History is a style reference only, never evidence of what the current change does.

## Scope of action

- **Draft / suggest a message**: reply with exactly one message as plain text — no preamble, quotes, or code fence — unless the user asks for alternatives or an explanation. Do not touch the repo.
- **Review a message**: give a verdict, the specific problems, and a corrected message.
- **Commit**: stage or commit only when the user explicitly asks. If the request is ambiguous, draft instead.
- **Amend**: only when explicitly asked, and only `HEAD`. Older commits need a rebase, which requires its own approval.
- Never push, change Git config, use `--no-verify`, or rewrite pushed history unless asked. Never force-push `main`/`master`.

## Message format

```text
<type>(<scope>)!: <description>

<body>

<footer>
```

### Type

| Type | Use for |
| --- | --- |
| `feat` | New user- or developer-facing capability (SemVer MINOR) |
| `fix` | Bug fix (SemVer PATCH) |
| `perf` | Performance improvement |
| `refactor` | Code change with no behavior change |
| `docs` | Documentation only |
| `test` | Adding or fixing tests |
| `build` | Build system, packaging, dependencies |
| `ci` | CI configuration and workflows |
| `style` | Formatting only (whitespace, semicolons); not UI styling |
| `revert` | Reverting a previous commit |
| `chore` | Maintenance that fits nothing above — last resort |

Pick the type by the change's effect, not its file paths: a dependency bump that fixes a bug is `fix`; tests and docs that accompany a feature stay in `feat`.

### Subject line (`<type>(<scope>)!: <description>`)

- **Length**: the whole line, prefix included. Aim for ≤ 50 characters (git's own recommendation); never exceed 72 — GitHub's web UI truncates longer subjects with "…". A repo rule such as commitlint's `header-max-length` (100 in `config-conventional`) overrides both.
- Imperative mood: "add", "fix", "remove" — not "added" or "adds". Test: "If applied, this commit will _<description>_".
- Start the description lowercase (commitlint rejects sentence-case by default); no trailing period.
- Describe the outcome, not the activity: `fix(auth): reject expired refresh tokens`, not `fix: update auth.ts`.
- Scope is optional. Use it only when a module or area is clearly identifiable from the diff or repo convention; omit it rather than guess.
- Language: follow the repo's history; with no history, use English unless the user asks otherwise.

### Body

Add one only when the subject can't carry the *why*: motivation, the previous behavior, trade-offs, side effects. Separate it from the subject with a blank line, wrap at 72 characters, and don't list changed files — the diff already does.

### Footer

- Breaking change: add `!` before the colon and/or a `BREAKING CHANGE: <what breaks and how to migrate>` footer. Valid on any type. Never claim a breaking change just to pass a lint rule.
- Issue references (`Refs: #123`, `Closes #123`) only for issues the user gave you or that appear in the context — never invent numbers.
- Add `Co-authored-by:`, `Signed-off-by:` or other trailers only when asked or required by the repo (e.g. DCO).
- Revert: `revert: <original subject>` with body `This reverts commit <full-sha>.`

### Examples

```text
feat(api): add cursor pagination to /orders
```

```text
fix(sync): retry uploads after token refresh

Uploads started just before token expiry failed with 401 and were
dropped silently. Refresh the token once and replay the request.
```

```text
refactor(config)!: drop support for YAML config files

BREAKING CHANGE: config.yaml is no longer read. Convert it to
config.toml; `app migrate-config` does this automatically.
```

## Evidence

- If the user supplied a diff, summary, or message, work from that; read the repo only for missing context or conventions.
- Otherwise the target is the **staged** snapshot: `git diff --staged`. Unstaged changes and branch names are not evidence of what is being committed.
- If the user asked about staged changes and nothing is staged, say so — don't silently fall back to the working tree.
- When evidence is thin, write the most conservative accurate description. Don't invent motivation or impact.
- Check conventions with `git log --oneline -15` and any commitlint config (`commitlint.config.*`, `.commitlintrc*`, `package.json#commitlint`).

## Committing

1. Inspect: `git status --short`, `git diff --staged`, `git diff`. Treat partially staged files as two separate snapshots.
2. Stage only what the user asked for. Keep an existing staged snapshot intact; if it contains out-of-scope changes, ask before unstaging. Prefer `git add <path>` over `git add -A`; never stage secrets, keys, `.env` files, or unrelated build output. Check untracked files before adding them.
3. One logical change per commit. If the scope mixes unrelated changes, suggest splitting, but don't split or drop anything without approval.
4. Run `git diff --staged --check`, then write the message from the final staged diff.
5. Commit with the message on stdin so the shell never reinterprets it:
   ```bash
   git commit --file=- <<'EOF'
   feat(api): add cursor pagination to /orders
   EOF
   ```
6. Let hooks run. If one fails, read its output, fix only in-scope problems, re-check the staged diff (hooks may modify files), and retry. Stop and report if the fix is out of scope or the same failure repeats.
7. Verify: confirm the exit status, then `git log -1 --format='%h %s'` and `git status --short`. Report the hash, subject, and anything left staged, unstaged, or untracked.

Don't use `--allow-empty` unless asked; with nothing staged, report that there is nothing to commit.

### Amending

- Message only: `git commit --amend --only --file=-` (no paths) keeps the tree and any staged changes untouched.
- Adding changes: stage them, then `git commit --amend --file=-`. The message must describe the whole resulting commit, not just the new part.
- If `HEAD` is already pushed, warn that amending requires a force-push before doing it.

## commitlint errors

Fix exactly what the reported rule (e.g. `subject-case`, `header-max-length`, `type-enum`) complains about, guided by the repo's actual config, and preserve the message's meaning. Validate with the repo's own entry point (e.g. `npx --no-install commitlint < msg.txt`), not by creating trial commits. Don't install packages, edit the config, or disable rules. Look up unfamiliar rules in the [rule reference](https://commitlint.js.org/reference/rules.html).
