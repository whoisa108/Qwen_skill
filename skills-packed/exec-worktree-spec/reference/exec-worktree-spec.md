# Exec Worktree Spec — Reference

---

## Table of Contents

1. [Git Worktree Command Reference](#1-git-worktree-command-reference)

---

## 1. Git Worktree Command Reference

Complete command reference for managing Git worktrees.

### Core Commands

#### Create

```bash
# New branch + worktree in one command (most common)
git worktree add -b <branch> <path> <base>

# Example:
git worktree add -b feat/oauth-login ../myapp-feat-oauth-login main

# Existing branch (no -b flag)
git worktree add <path> <existing-branch>

# Detached HEAD at specific commit
git worktree add --detach <path> <commit-hash>
```

#### Inspect

```bash
git worktree list           # paths, branches, HEAD commits
git worktree list --verbose # extended info
git worktree list --porcelain  # machine-readable (for scripting)
```

Sample output:
```
/home/user/myapp           abc1234 [main]
/home/user/myapp-feat-auth def5678 [feat/oauth-login]
/home/user/myapp-fix-nav   ghi9012 [fix/nav-crash]
```

#### Remove

```bash
git worktree remove <path>          # clean (fails if uncommitted changes exist)
git worktree remove --force <path>  # force (discards uncommitted changes)
git worktree prune                  # prune stale entries after manual deletion
```

#### Move / Repair

```bash
git worktree move <path> <new-path>   # move to new location
git worktree repair <path>            # repair broken link after manual move
```

---

### Workflow Patterns

**Feature isolation (most common):**
```bash
git worktree add -b feat/payments ../myapp-feat-payments main
cd ../myapp-feat-payments
# ... develop ...
cd ../myapp
git worktree remove ../myapp-feat-payments
git branch -d feat/payments
```

**Hotfix while feature in progress:**
```bash
git worktree add -b fix/login-crash ../myapp-fix-login-crash main
cd ../myapp-fix-login-crash
# ... fix, commit, push PR ...
cd ../myapp
git worktree remove ../myapp-fix-login-crash
```

**Review a PR locally:**
```bash
git fetch origin
git worktree add ../myapp-review-pr-42 origin/feat/their-feature
# ... review, run tests ...
git worktree remove ../myapp-review-pr-42
```

**Parallel agent sessions:**
```bash
git worktree add -b feat/auth     ../myapp-agent-auth     main
git worktree add -b feat/payments ../myapp-agent-payments main
# Each agent operates in its own directory — no conflicts
```

---

### Constraints and Limitations

| Constraint | Detail |
|---|---|
| One checkout per branch | Cannot check out the same branch in two worktrees |
| Shared `.git` | All worktrees share the same object store and config |
| Hooks apply to all | `.git/hooks/` fires in all worktrees |
| No nested worktrees | Cannot create a worktree inside another worktree's directory |
| Submodules | NOT automatically initialized — run `git submodule update --init` |

---

### Naming Conventions

| Convention | Pattern | Example |
|---|---|---|
| Sibling directory (recommended) | `../<repo>-<branch-slug>` | `../myapp-feat-oauth` |
| Worktrees subdirectory | `./.worktrees/<branch>` | `./.worktrees/feat-oauth` |
| Temp review | `../review-<pr-number>` | `../review-pr-142` |

---

### Troubleshooting

**"fatal: '<path>' already exists"**
```bash
rm -rf <path>          # if safe to delete
git worktree prune     # if path was manually deleted
```

**"fatal: '<branch>' is already checked out at '<path>'"**
```bash
git worktree list
git worktree remove <conflicting-path>
```

**Submodules not present in new worktree:**
```bash
cd <new-worktree-path>
git submodule update --init --recursive
```
