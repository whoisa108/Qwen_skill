---
name: git-worktree-design
description: Analyze a multi-feature or parallel development request and design a Git worktree strategy — proposing how to split work across branches and worktrees before any execution. Automatically trigger this skill when the user mentions "worktree", "git worktree", "parallel branches", "多分支開發", "parallel development", or when the request involves building multiple independent features simultaneously, or when the user describes a roadmap or sprint with multiple unrelated tasks that could benefit from isolation. Trigger BEFORE exec-worktree-spec — this skill designs the plan, that skill executes it.
---

# Git Worktree Design

Design a parallel development strategy using Git worktrees. This skill produces a **confirmed plan** — execution is handled by `exec-worktree-spec`.

---

## Step 1 — Understand the Scope

Gather context before proposing anything:

```bash
git branch -a              # existing branches
git worktree list          # existing worktrees
git log --oneline -10      # recent history, infer project conventions
ls                         # project structure
```

Ask if unclear:
- What are the individual features/tasks? (list them)
- Any dependencies between them? (can A merge before B?)
- Shared deadline or independent timelines?
- Target branch for each (all to `main`, or different)?

---

## Step 2 — Analyze Dependencies

Map each feature to a dependency class:

| Class | Definition | Worktree Strategy |
|---|---|---|
| **Independent** | No shared files, no order requirement | Full parallel — separate worktrees |
| **Sequential** | B depends on A's output | Chain — B branches from A |
| **Partially shared** | Overlap in 1-2 files | Parallel with merge plan noted |
| **Tightly coupled** | Deep shared state | Recommend single branch instead |

Identify dependency class for each pair. Quote which files would conflict if parallel: `file:line`.

---

## Step 3 — Propose Worktree Plan

Present a concrete plan with explicit paths and branch names:

```
Proposed Worktree Strategy
==========================

Base branch: main (abc1234)

Worktree 1 — feat/user-auth
  Path:    ../myapp-feat-user-auth
  Scope:   src/auth/, src/middleware/
  Depends: none (start immediately)

Worktree 2 — feat/dashboard-ui
  Path:    ../myapp-feat-dashboard-ui
  Scope:   src/components/dashboard/, src/styles/
  Depends: none (start immediately)

Worktree 3 — feat/api-rate-limiting
  Path:    ../myapp-feat-api-rate-limiting
  Scope:   src/api/, config/nginx.conf
  Depends: feat/user-auth must merge first (shared: src/api/auth-check.ts)

Merge order: 1 → 3 → 2 (2 is fully independent, can merge any time)

Potential conflict zone: src/api/router.ts modified by both 1 and 3
  → Recommend: keep route registration in separate files per feature
```

---

## Step 4 — Confirm With User

Present the plan and explicitly ask:
1. Does the scope split match your intent?
2. Any features missing or should be merged together?
3. Confirm merge order if dependencies exist.

Do not proceed to execution until user confirms.

---

## Step 5 — Hand Off to Execution

Once confirmed, state clearly:

> "Plan confirmed. I'll now use `exec-worktree-spec` to create each worktree."

Then execute each worktree creation per the plan, referencing confirmed branch names and paths.

---

## Design Principles

**Prefer narrow scope per worktree:** Each worktree should touch a distinct set of files. If two features share > 3 files, flag it.

**Name branches from the task:** `feat/<noun-verb>` — derive from the feature description, not from the tech (`feat/oauth-login`, not `feat/jwt-changes`).

**Sibling directory convention:** Place worktrees as `../<repo>-<branch-slug>` to keep them discoverable and out of the main repo directory.

**Document the conflict zone:** Always explicitly list files that appear in multiple worktrees. This is the primary risk surface.

---

## Edge Cases

**User requests > 5 parallel worktrees:**
Flag cognitive overhead. Suggest prioritizing 2-3 active worktrees; others can be created when current ones merge.

**Monorepo with shared config files:**
`package.json`, `tsconfig.json`, etc. are high-conflict risk. Note them explicitly. Recommend config changes go in a dedicated `chore/config-*` branch that merges first.

**User hasn't decided on features yet:**
Switch to discovery mode: ask "What problems are you trying to solve?" then derive feature split from answers.

**Single-developer project:**
Note that worktrees add overhead for solo work. Recommend only if tasks are genuinely independent and long-running (> 1 day each).

---

## References

Read **`reference/git-worktree-design.md`** when:

- **§1 Worktree Commands** — explaining worktree mechanics to the user, or a specific pattern (hotfix isolation, PR review, parallel agents) needs command-level detail.
- **§2 Merge Conflict Strategy** — two or more proposed worktrees share files, user asks about merge order, or you need conflict risk classification, pre-branch refactor pattern, merge order diagrams, or the `comm` command to detect shared files between branches.
