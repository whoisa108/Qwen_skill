# Skill Development — Reference

---

## Table of Contents

1. [Description Optimization](#1-description-optimization)
2. [Body Writing Patterns](#2-body-writing-patterns)

---

## 1. Description Optimization

Patterns for writing skill descriptions that trigger reliably.

### How Triggering Works

Claude reads `name` + `description` metadata at the start of every conversation. It asks: "Would this skill help me respond better?" Claude **undertriggers by default** — descriptions must make the case that consulting the skill is worth it.

### Description Structure Template

```
[One-sentence summary of what the skill does.]
[When to trigger: explicit actions, phrases, or contexts.]
[Synonyms and alternate phrasings to catch.]
[Non-obvious trigger cases ("Also trigger when...").]
[Optional: what this skill does NOT handle.]
```

### Trigger Phrase Patterns

```
Trigger this skill when the user wants to [verb phrase]...
Use when the user says [exact phrase] or [synonym]...
Automatically activate when the request involves [pattern]...
...mentions "worktree", "git worktree", "parallel branches", "多分支開發"...
...when the user describes a multi-feature roadmap or sprint...
```

### Good vs Bad Examples

**Too narrow (undertriggers):**
```yaml
description: "Create a Pull Request description."
```
Misses: "write PR", "draft PR body", "prepare for review", "what changed in this branch"

**Too broad (overtriggers):**
```yaml
description: "Help with Git. Use for any Git-related task."
```
Would trigger on "git init", "explain rebasing" — tasks it can't actually help with.

**Just right:**
```yaml
description: >
  Generate a Pull Request description by analyzing the diff between the current
  branch and a target branch. Trigger when the user wants to create, write, or
  draft a PR/MR description, open a pull request, summarize branch changes for
  review, or mentions "PR template", "pull request body", "merge request
  description". Also trigger when the user asks to "prepare this branch for
  review" or "what changed in this branch".
```

### Disambiguation Clauses

When two skills cover adjacent domains, add explicit "NOT for X" to each:

```yaml
# In git-worktree-design:
description: "...design a worktree strategy. NOT for executing worktree
  creation — use exec-worktree-spec for that."

# In exec-worktree-spec:
description: "...execute worktree creation. For upfront planning and
  branch strategy design, use git-worktree-design instead."
```

### Pushiness Calibration

| Level | Example | When to use |
|---|---|---|
| Neutral | "Use when X" | High-specificity skills with unique triggers |
| Pushy | "Automatically trigger when..." | Skills Claude often skips |
| Very pushy | "Always use this when Y, even if not explicitly asked for Z" | Skills sharing keywords with simpler tasks |

### Length Guidelines

| Length | Risk |
|---|---|
| < 30 words | Undertriggering — too few signal words |
| 30–100 words | Sweet spot |
| > 150 words | Context overhead; trim synonyms |

### Mental Test Framework

Before finalizing, generate 5 should-trigger and 5 should-NOT-trigger prompts:

**Should trigger (varied phrasings):**
1. Obvious: "write a PR description for my branch"
2. Synonym: "draft the pull request body"
3. Indirect: "I need to prepare this for code review"
4. Casual: "can you write up what changed so I can open the PR"
5. Non-English (if applicable): "幫我寫 PR 說明"

**Should NOT trigger (near-misses):**
1. "explain what a pull request is" (educational)
2. "review my PR description" (reviewing, not generating)
3. "merge this branch" (different Git operation)
4. "what's the diff between these two files" (diff inspection)
5. "git help" (generic)

---

## 2. Body Writing Patterns

Structural patterns for skill bodies that produce consistent outputs.

### The Evidence-First Rule (QWEN Core)

```
1. Observe   — read files, run diagnostic commands, gather state
2. Analyze   — find patterns, group concerns, identify risks
3. Propose   — present plan with file:line citations
4. Confirm   — wait for user approval on non-trivial actions
5. Execute   — run commands or write files
6. Verify    — confirm result matches intent
```

**Never skip to Execute without completing Observe + Analyze.**

### Pattern A — Decision Gate

Use when the skill might not be needed for every trigger.

```markdown
## Should I Run This Skill?

Run if ANY of the following are true:
- [condition 1]
- [condition 2]

Skip and explain if:
- [exclusion condition]

Evidence to collect first:
\```bash
<diagnostic commands>
\```
```

### Pattern B — Propose-Confirm-Execute

Use for any action that modifies files, branches, or state.

```markdown
## Step N — Propose [Action]

Present the plan:
\```
[formatted plan output]
\```

Ask: "[Specific yes/no question]"
Do not proceed until user confirms.

## Step N+1 — Execute

After confirmation:
\```bash
<exact commands>
\```
```

### Pattern C — Grouped Output

Use when generating structured content (commits, PR descriptions, etc.).

```markdown
## Output Format

Group by [concern/feature/type]:

\```
Group 1 — [Label]
  [items with file:line citations]

Group 2 — [Label]
  [items with file:line citations]
\```

Present before executing.
```

### Pattern D — Branching Logic

Use when behavior depends on what the agent finds.

```markdown
## Step N — Check for [Condition]

\```bash
<detection command>
\```

**If found:** [action A]
**If not found:** [action B, or default behavior]
**If ambiguous:** Ask: "[specific question]"
```

### Pattern E — Cleanup + Remind

Use for skills that create temporary state.

```markdown
## Cleanup

When task is complete, remind the user:
\```bash
<cleanup commands>
\```
Note: [when these are safe to run]
```

### Command Writing Rules

**Always use concrete, copyable commands:**

```bash
# Good — executable as-is
git diff origin/main...HEAD --stat

# Bad — requires mental substitution
git diff <target>..<current> --stat
```

Use `<placeholder>` only for values the agent must derive at runtime.

### Edge Case Format

Each edge case must have:
1. Concrete trigger condition
2. Specific response action
3. Exact command or message template

```markdown
**[Edge case name]:**
Trigger: [exact condition observed]
Action: [what to do — exact command or message]
```

**Example:**
```markdown
**Branch name collision:**
Trigger: `git worktree add` fails with "already exists"
Action: Append `-2` suffix and retry once. If still fails:
  `Branch <n> and <n>-2 both exist. Please specify a unique name.`
```

### File:Line Citation Rules

Cite `file:line` when:
- Grouping changes by concern
- Proposing a plan that touches specific code sections
- Noting conflict risk in a shared file

Format: `path/to/file.ts:42` or `path/to/file.ts:42-78`

Never cite a file without a line number unless the entire file is relevant.

### Skill Handoff Declaration

When handing off to another skill:

```markdown
## Handoff to [skill-name]

This skill's job is complete. Next step: [what needs to happen].

> "I'll now use `[skill-name]` to [specific action]."

Carry forward: [what information from this skill's output is needed next]
```
