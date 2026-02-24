# Git Smart Commit — Reference

---

## Table of Contents

1. [Conventional Commits Specification](#1-conventional-commits-specification)
2. [Interactive Staging (`git add -p`)](#2-interactive-staging-git-add--p)

---

## 1. Conventional Commits Specification

Full specification for writing atomic, meaningful commit messages.
Source: https://www.conventionalcommits.org/en/v1.0.0/

### Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Rules:**
- `type` and `description` are mandatory
- `scope` is optional; use the module/domain name in parentheses
- Description: imperative, present tense, no capital first letter, no period at end
- Body: explain *why*, not *what* (the diff already shows what). Wrap at 72 chars.
- Footer: reference issues (`Fixes #123`), note breaking changes (`BREAKING CHANGE: ...`)

### Type Reference

| Type | When to use | Example |
|---|---|---|
| `feat` | New feature for the user | `feat(auth): add OAuth2 login` |
| `fix` | Bug fix for the user | `fix(api): handle null response from payment gateway` |
| `docs` | Documentation only | `docs(readme): update installation steps` |
| `style` | Formatting, whitespace — no logic change | `style: apply prettier formatting` |
| `refactor` | Code restructure — no feature or fix | `refactor(cart): extract price calculation to service` |
| `test` | Adding or fixing tests | `test(auth): add edge cases for token expiry` |
| `chore` | Build process, tooling, dependencies | `chore(deps): upgrade axios to v1.6` |
| `perf` | Performance improvement | `perf(query): add index on user.email` |
| `ci` | CI/CD configuration changes | `ci: add lint step to pull_request workflow` |
| `revert` | Reverting a previous commit | `revert: feat(auth): add OAuth2 login` |

### Scope Guidelines

Scope should reflect the **domain or module** affected, not the file name:

| Good scope | Bad scope |
|---|---|
| `auth` | `authController` |
| `cart` | `cart.service.ts` |
| `api` | `routes` |
| `deps` | `package` |

Omit scope when change is truly cross-cutting (e.g., global config changes).

### Breaking Changes

Mark breaking changes in the footer with `BREAKING CHANGE:` keyword, OR by appending `!` after type/scope:

```
feat(api)!: change response envelope from `data` to `result`

BREAKING CHANGE: All API consumers must update response destructuring.
Before: const { data } = response
After:  const { result } = response
```

### Body Example

```
fix(payments): retry failed webhook delivery on 5xx errors

Previously, webhook failures were silently dropped, causing missed
payment confirmations. Now retries up to 3 times with exponential
backoff (1s, 2s, 4s) before marking as failed.

Affects: src/webhooks/delivery.ts:45-78
Related to customer reports: #234, #251
```

### Atomic Commit Checklist

A commit is atomic if:
- [ ] It does exactly ONE logical thing
- [ ] It passes CI on its own (if checked out in isolation)
- [ ] Its description fully explains what changed without reading the diff
- [ ] Reverting it doesn't break unrelated functionality

### Common Anti-patterns

| Anti-pattern | Instead |
|---|---|
| `fix: various bugs` | Separate `fix` per bug |
| `feat: wip` | Finish the feature or use a draft branch |
| `chore: update stuff` | Name what was updated |
| `fix: pr comments` | Describe the actual change made |
| Giant commit with 20 files | Split by `feat`/`fix`/`test`/`chore` |

---

## 2. Interactive Staging (`git add -p`)

Use when a single file contains changes belonging to multiple logical commits.

### When to Use

- One file modified for two different features
- Bug fix mixed with refactoring in the same file
- Config change mixed with dependency update in `package.json`

### Basic Flow

```bash
git add -p <file>
# or scan all files:
git add -p
```

Git splits the file into **hunks** (contiguous changed sections) and prompts for each:

```
@@ -45,7 +45,12 @@ function authenticate(user) {
+  if (!user.email) throw new ValidationError('email required')
+  if (!user.password) throw new ValidationError('password required')
   return db.users.findOne({ email: user.email })
```

### Hunk Commands

| Key | Action | When to use |
|---|---|---|
| `y` | Stage this hunk | Belongs to current commit |
| `n` | Skip this hunk | Belongs to a later commit |
| `s` | Split into smaller hunks | Hunk mixes two concerns |
| `e` | Manually edit hunk | Need surgical precision |
| `q` | Quit | Done staging for this commit |
| `?` | Show help | — |
| `d` | Skip file entirely | Whole file belongs to later commit |
| `a` | Stage all remaining hunks in file | Rest of file is for this commit |

### Splitting Workflow Example

Scenario: `src/auth/user.ts` has both a validation fix and a new logging feature.

```bash
# First commit: just the validation fix
git add -p src/auth/user.ts
# → y for validation hunk, n for logging hunk
git commit -m "fix(auth): validate email format before DB lookup"

# Second commit: the logging feature
git add src/auth/user.ts
git commit -m "feat(auth): add structured login audit logging"
```

### Manual Edit Mode (`e`)

When `s` (split) isn't granular enough:

```diff
# To unstage an addition: change `+` to ` ` (space)
# To unstage a deletion: remove the `-` line entirely (risky — avoid)
```

### Verify Before Committing

```bash
git diff --cached    # exactly what will be committed
git diff             # what remains unstaged
```

### Reset Staged Changes

```bash
git restore --staged <file>   # unstage one file
git restore --staged .        # unstage everything
```
