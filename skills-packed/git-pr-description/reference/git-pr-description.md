# Git PR Description — Reference

---

## Table of Contents

1. [PR Description Writing Guide](#1-pr-description-writing-guide)

---

## 1. PR Description Writing Guide

Patterns and examples for writing high-quality Pull Request descriptions.

### Anatomy of a Good PR Description

A PR description answers four questions for the reviewer:

1. **What** changed? (Changes section)
2. **Why** was this change needed? (Summary / motivation)
3. **How** was it tested? (Test Plan)
4. **What** should the reviewer focus on? (optional Review Focus)

---

### Summary Section Patterns

The summary should state **intent and motivation**, not echo the title.

**Weak:**
> This PR modifies the authentication module and updates several files.

**Strong:**
> Users were being logged out unexpectedly when their session token expired mid-request. This adds silent token refresh before expiry to keep sessions alive without requiring re-login.

**Extracting intent from diff signals:**

| Diff signal | Likely intent |
|---|---|
| New validation before DB call | Input safety / preventing bad state |
| New `try/catch` around external call | Resilience / error handling |
| Extracting method from large function | Readability / testability |
| New index on DB column | Query performance |
| Changing response shape | API contract change — flag explicitly |

---

### Changes Section Patterns

Group by **concern**, not by file. Use present tense imperatives.

**By feature layer:**
```markdown
## Changes

**API**
- Add `POST /auth/refresh` endpoint for silent token renewal
- Return 401 with `reason: token_expired` instead of generic 401

**Client**
- Intercept 401 responses and attempt token refresh before retry
- Redirect to login only after refresh also fails
```

**By concern (single-layer):**
```markdown
## Changes

**Validation**
- Reject requests where `email` is missing or malformed (`src/auth/validator.ts:34`)
- Add error code `AUTH_001` for missing credentials

**Error handling**
- Wrap DB lookup in try/catch; surface as 503 on connection failure
```

**What NOT to do:**
```markdown
# BAD — file-by-file listing
- Updated auth.controller.ts
- Modified user.service.ts
- Changed index.ts
```

---

### Test Plan Patterns

```markdown
## Test Plan

- [x] Unit: `AuthService.refreshToken` — 6 new cases covering expiry boundary,
      invalid signature, revoked token (`src/auth/__tests__/token.test.ts`)
- [x] Integration: login → expire → refresh → protected route access
- [x] Manual: confirmed no logout occurs after token expiry
- [ ] Load test: not done — endpoint is not on critical path
```

**When no tests exist in the diff:**
```markdown
## Test Plan

No automated tests added. Verified manually:
- Reproduced the original bug (step: ...)
- Confirmed fix resolves it (step: ...)

TODO: unit tests for `refreshToken` should follow in #456
```

---

### Common PR Template Fields

| Section | Fill with |
|---|---|
| `## Description` | Summary: what + why |
| `## Type of change` | Check matching box; uncheck others |
| `## How has this been tested?` | Specific test cases or manual steps |
| `## Checklist` | Check honestly; leave unchecked if not done |
| `## Related Issues` | `Closes #123` or `Related to #456` |

**`Closes` vs `Related to`:**
- `Closes #n` / `Fixes #n` — issue auto-closes on merge
- `Related to #n` — connected but not fully resolved

---

### Breaking Change Callout

```markdown
## ⚠️ Breaking Change

`GET /users/:id` response shape changed:

Before: `{ user: { id, name, email } }`
After:  `{ data: { id, name, email }, meta: { version } }`

All clients must update response destructuring before this merges.
Tracking migration: #789
```

---

### Review Focus (optional)

```markdown
## Review Focus

- `src/auth/token.ts:67-89` — refresh timing logic; please verify the
  expiry window calculation (`exp - 60s` — unsure if too aggressive)
- `migrations/0042_add_refresh_tokens.sql` — confirm column types match ORM model
```
