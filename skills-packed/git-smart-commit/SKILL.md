---
name: git-smart-commit
description: Intelligently split messy or mixed staged/unstaged changes into multiple meaningful Conventional Commits, each scoped to a single logical concern. Trigger this skill when the user wants to commit changes but has mixed or unrelated modifications, asks to "split commits", "organize commits", "clean up changes before committing", "make atomic commits", or says something like "I have a bunch of changes, help me commit them properly". Also trigger when the user just says "commit my changes" and the working tree has modifications spanning multiple concerns.
---

# Git Smart Commit

Transform disorganized working tree changes into clean, atomic Conventional Commits. Read the actual diff — never assume what changed.

---

## Step 1 — Snapshot Current State

```bash
git status --short
git diff --stat          # unstaged
git diff --cached --stat # staged
```

Identify: total changed files, which are staged vs unstaged, any untracked files.

---

## Step 2 — Read the Full Diff

```bash
git diff HEAD
```

For each changed file, note:
- **What** changed (logic, config, style, test, docs)
- **Why** it likely changed (infer from context + surrounding code)
- **Which feature/concern** it belongs to

Reference as `file:line` when grouping.

---

## Step 3 — Propose Groupings

Group files by **logical concern**, not by directory or file type.

**Grouping heuristics:**
| Signal | Group together |
|---|---|
| Same feature touched across layers | `feat/fix` scope |
| Only test files | `test` type |
| Only comments/docs | `docs` type |
| Only formatting (whitespace, indent) | `style` type |
| Config/tooling changes | `chore` type |
| Unrelated bug fix found alongside feature | Separate `fix` commit |

Present the plan **before executing**:

```
Proposed commits (3):

1. feat(auth): add JWT refresh token support
   Files: src/auth/token.ts:12-45, src/auth/middleware.ts:8-22

2. test(auth): add unit tests for token refresh logic
   Files: src/auth/__tests__/token.test.ts

3. chore(deps): upgrade jsonwebtoken to v9
   Files: package.json, package-lock.json
```

Ask: "Does this grouping look right? Any changes before I execute?"

---

## Step 4 — Execute Commits

After user confirms, execute each commit using selective staging:

```bash
# Stage specific files
git add <file1> <file2>

# Stage specific hunks (if one file spans multiple concerns)
git add -p <file>

# Commit with conventional message
git commit -m "<type>(<scope>): <description>"
```

**Conventional Commit format:**
```
<type>(<scope>): <short description>

[optional body: explain WHY, not WHAT — reference file:line for non-obvious changes]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`

---

## Step 5 — Verify

```bash
git log --oneline -<n>   # n = number of commits made
git status               # should be clean
```

Report:
```
✓ 3 commits created:
  abc1234 feat(auth): add JWT refresh token support
  def5678 test(auth): add unit tests for token refresh logic
  ghi9012 chore(deps): upgrade jsonwebtoken to v9

Working tree: clean
```

---

## Edge Cases

**One file touches two concerns:**
Use `git add -p` to stage specific hunks. Explain which hunks go to which commit.

**Untracked files:**
List separately. Ask user whether to include (some may be build artifacts).

**Already partially staged:**
Note what's staged vs unstaged. Ask whether to incorporate existing staging or start fresh.

**Whitespace-only changes mixed in:**
Isolate into a `style` commit. Never mix style with logic changes.

**Cannot determine scope:**
Use the top-level directory name as scope, or omit scope entirely. Do not guess.

---

## Commit Message Quality Rules

- Description: imperative mood, < 72 chars, no period
- Body: explain *why* the change was needed, not *what* it does (the diff shows what)
- No "WIP", "misc", "various fixes", "update stuff"
- If a change fixes a bug found during feature work, it gets its own `fix` commit

---

## References

Read **`reference/git-smart-commit.md`** when:

- **§1 Conventional Commits** — unsure which `type` to use, writing a commit body with breaking changes, or user asks "what's the right commit format for X". Contains full type table, scope guidelines, body examples, anti-patterns.
- **§2 Interactive Staging (`git add -p`)** — a single file contains changes from two different concerns that need to be split across commits. Contains hunk commands, edit mode, splitting workflow example.
