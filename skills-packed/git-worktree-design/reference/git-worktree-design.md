# Git Worktree Design — Reference

---

## Table of Contents

1. [Git Worktree Command Reference](#1-git-worktree-command-reference)
2. [Merge Conflict Strategy](#2-merge-conflict-strategy)

---

## 1. Git Worktree Command Reference

### Core Commands

```bash
# Create: new branch + worktree
git worktree add -b <branch> <path> <base>

# Create: existing branch
git worktree add <path> <existing-branch>

# List
git worktree list
git worktree list --verbose

# Remove
git worktree remove <path>
git worktree prune        # clean up stale entries

# Move / Repair
git worktree move <path> <new-path>
git worktree repair <path>
```

### Naming Convention

Use sibling directories: `../<repo>-<branch-slug>`

```
../myapp-feat-auth
../myapp-feat-payments
../myapp-fix-nav-crash
```

### Constraints

| Constraint | Detail |
|---|---|
| One checkout per branch | Cannot use same branch in two worktrees |
| Shared `.git` | All worktrees share object store and config |
| No nested worktrees | Cannot create worktree inside another worktree |
| Submodules | Run `git submodule update --init` after creation |

---

## 2. Merge Conflict Strategy

### Conflict Risk Classification

Before creating parallel worktrees, classify each shared file:

| Risk Level | Signal | Strategy |
|---|---|---|
| **Critical** | Shared entry points (`index.ts`, `app.ts`, `router.ts`) | Serialize — one branch merges first |
| **High** | Shared config (`package.json`, `tsconfig.json`, `.env.example`) | Dedicated `chore/config` branch, merges first |
| **Medium** | Shared utility functions used by both features | Extract to new shared module before branching |
| **Low** | Same directory, different files | Full parallel — Git merges cleanly |
| **None** | Completely separate directories | Full parallel — no risk |

### Pre-branch Refactor Pattern

When two features both need to modify a shared module, refactor first:

```
Before branching:
  shared/utils.ts  ← both features need to modify this

Strategy:
  1. On main: extract into utils/auth.ts + utils/payments.ts
  2. Commit: refactor(utils): split utils into domain-specific modules
  3. Now branch — each feature touches only its own utils file
```

This is cheaper than resolving conflicts later.

### Merge Order Diagrams

**Independent features (no shared files):**
```
main ──●──────────────────────────────●── merge A
        \──── feat/A ────────────────/
        \──── feat/B ───────────────────●── merge B
```
Merge in any order.

**Sequential dependency (B needs A's output):**
```
main ──●──────────────●── merge A ──────────────●── merge B
        \──── feat/A ─/                           \
                       \──── feat/B (from A) ──────/
```
Branch B from A, not from main. Rebase B onto main after A merges.

**Shared config file:**
```
main ──●── chore/config ──●── merge ──────────●── merge A ──●── merge B
                                     \── feat/A ──/    \── feat/B ──/
```
Config branch merges first. Feature branches rebase onto updated main.

### Rebase After Parallel Merge

When feature B is in progress after feature A merges:

```bash
git fetch origin
git rebase origin/main

# On conflict:
git status
# resolve each file...
git add <resolved-file>
git rebase --continue
```

**Never merge main into a feature branch** if the repo uses rebase strategy. Confirm with user first.

### Common Conflict Resolutions

**`package.json` dependency conflict:**
```json
<<<<<<< HEAD
  "axios": "^1.5.0",
=======
  "axios": "^1.6.2",
  "jsonwebtoken": "^9.0.0",
>>>>>>> feat/auth
```
Take the higher version of shared deps; add new deps from feature branch.

**Import conflict in entry files:**
```typescript
<<<<<<< HEAD
import { CartRouter } from './cart/router'
=======
import { AuthRouter } from './auth/router'
>>>>>>> feat/auth
```
Keep both imports — they're additive.

### Conflict Prevention Checklist

Before creating parallel worktrees:

```bash
# Find files changed by each branch
git diff main...feat/A --name-only > /tmp/files-a.txt
git diff main...feat/B --name-only > /tmp/files-b.txt

# Find intersection (shared files)
comm -12 <(sort /tmp/files-a.txt) <(sort /tmp/files-b.txt)
```

- [ ] Classify each shared file by risk level (see table above)
- [ ] Plan pre-branch refactor for Medium-risk shared modules
- [ ] Decide merge order for Critical/High-risk files
- [ ] Document merge order in worktree design plan
- [ ] Note in each PR: "Must merge after #X" or "Safe to merge independently"
