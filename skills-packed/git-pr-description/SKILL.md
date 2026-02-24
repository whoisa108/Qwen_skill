---
name: git-pr-description
description: Generate a Pull Request description by analyzing the diff between the current branch and a target branch. Trigger this skill when the user wants to create, write, or draft a PR/MR description, open a pull request, summarize branch changes for review, or mentions "PR template", "pull request body", "merge request description". Also trigger when the user asks to "prepare this branch for review" or "what changed in this branch".
---

# Git PR Description

Generate accurate, evidence-based Pull Request descriptions from actual diff data. Never summarize from memory — always read the diff first.

---

## Step 1 — Establish Branches

```bash
git branch --show-current          # current (source) branch
git remote show origin | grep HEAD  # default target (usually main/master)
```

Ask user to confirm target branch if ambiguous. Default: `main`.

---

## Step 2 — Collect Evidence

Run all three; use the output as your primary source:

```bash
# 1. Commit list with messages
git log <target>..<current> --oneline --no-merges

# 2. File-level change summary
git diff <target>...<current> --stat

# 3. Full diff (for understanding intent)
git diff <target>...<current>
```

For large diffs (> 500 lines), read `--stat` first and selectively read relevant hunks.

---

## Step 3 — Check for PR Template

```bash
ls .github/pull_request_template.md \
   .github/PULL_REQUEST_TEMPLATE.md \
   .github/PULL_REQUEST_TEMPLATE/*.md \
   docs/pull_request_template.md 2>/dev/null
```

**If template found:** Fill in every section of the template exactly. Do not add or remove sections.

**If no template:** Use the default structure below.

---

## Step 4 — Generate Description

### Default Structure (no template)

```markdown
## Summary
<!-- 2-3 sentences: what changed and why -->

## Changes
<!-- Group by concern, not by file. Use present tense. -->
- Add X to support Y
- Fix Z where condition W caused bug

## Test Plan
<!-- How was this verified? Be specific. -->
- [ ] Unit tests added for <module>:`<file>:line`
- [ ] Manual test: <specific scenario>

## Related
<!-- Issues, tickets, or prior PRs if detectable from commit messages -->
```

### Writing Rules

- **Summary**: State the *purpose*, not just "changed X file". Derive from commit messages + diff intent.
- **Changes**: Group logically (e.g., "Auth", "UI", "Config") — not file-by-file. Reference `file:line` for non-obvious changes.
- **Test Plan**: Extract from diff — look for `*.test.*`, `*.spec.*`, test directories. If none found, note "No tests found in diff" and suggest what to add.
- **Tone**: Active voice, present tense. "Add validation" not "Added validation".

---

## Edge Cases

**Empty diff (branches identical):**
Report: `No commits found between <target> and <current>. Confirm branch names.`

**Merge commits in log:**
Skip with `--no-merges`; note if only merge commits exist.

**Binary files in diff:**
List them separately: `Binary changes: [file list]`

**Template with unfamiliar placeholders:**
Fill what you can; mark unknowns with `<!-- TODO: fill in -->`.

---

## Output

Print the description as a markdown code block so the user can copy it directly. If the template was used, note which file it came from: `(using .github/pull_request_template.md)`.

---

## References

Read **`reference/git-pr-description.md`** when:

- **§1 PR Writing Guide** — generating a PR with multiple concerns, writing a breaking change notice, diff is large and needs careful grouping, or writing a Review Focus section. Contains summary patterns, Changes grouping examples, Test Plan templates, breaking change callout format.
