---
name: exec-worktree-spec
description: Execute and manage Git worktree creation based on situational analysis. Use this skill when the agent needs to autonomously decide whether to create a new worktree — for example, when a task involves long-running work that shouldn't block the main branch, when the user is starting a new feature that requires isolation, when parallel workstreams are detected, or when the current working tree is dirty and the task requires a clean environment. Automatically trigger this when context suggests worktree isolation would prevent conflicts or improve parallel progress.
---

# Exec Worktree Spec

Autonomously analyze the current situation and create Git worktrees when appropriate. This skill handles the **execution** phase — for upfront design and planning of worktree strategy, use `git-worktree-design` instead.

---

## Decision: Should I Create a Worktree?

Before acting, evaluate these signals:

**Create a worktree if ANY of these are true:**
- Current working tree has uncommitted changes unrelated to the task
- Task is long-running (> ~30 min estimated) and would block main branch
- Task requires a different base branch than current HEAD
- Another agent or process is actively using the current worktree
- User explicitly requests isolation

**Do NOT create a worktree if:**
- Task is a quick fix (< 5 files, < 15 min)
- Already inside a worktree for this task
- Repo has no existing commits (bare repo edge case)

Quote your evidence: `git status` output, branch name, task scope estimate.

---

## Execution Protocol

### Step 1 — Gather Context (always run first)

```bash
git status --short
git branch --show-current
git worktree list
```

Identify: current branch, dirty files, existing worktrees, and target branch for the task.

### Step 2 — Determine Worktree Spec

Derive the spec from task context:

| Field | Source |
|---|---|
| `branch_name` | Infer from task: `feat/`, `fix/`, `chore/` prefix + kebab-case description |
| `base_branch` | Default to `main` or `master`; use current branch if task is an extension |
| `path` | `../[repo-name]-[branch-name]` (sibling directory convention) |

Example derivation:
- Task: "Add OAuth login support"
- Branch: `feat/oauth-login`
- Path: `../myapp-feat-oauth-login`

### Step 3 — Execute

```bash
# Create branch + worktree in one command
git worktree add -b <branch_name> <path> <base_branch>
```

If branch already exists:
```bash
git worktree add <path> <branch_name>
```

### Step 4 — Verify and Report

```bash
git worktree list
```

Report with concrete references:
```
✓ Worktree created
  Path:   ../myapp-feat-oauth-login
  Branch: feat/oauth-login  (from main @ abc1234)
  Status: clean
```

---

## Edge Cases

**Dirty working tree on base branch:**
Stash changes before creating worktree, or warn user that uncommitted changes stay on original branch.

**Branch name collision:**
Append `-2`, `-3` suffix. Never silently overwrite.

**Path already exists:**
Abort and report: `[path] already exists — remove it first or choose a different branch name.`

**Detached HEAD:**
Create branch from current commit: `git worktree add -b <branch_name> <path> HEAD`

---

## Cleanup Reference

When the task is complete, remind user:
```bash
git worktree remove <path>
git branch -d <branch_name>  # only after merge
```

---

## References

Read **`reference/exec-worktree-spec.md`** when:

- **§1 Worktree Commands** — encountering an unusual worktree state (broken link, stale entry, submodule missing), needing `worktree repair` or `prune`, or user asks about a specific workflow pattern (hotfix, PR review, parallel agents). Contains full command reference, workflow patterns, constraints table, troubleshooting.
