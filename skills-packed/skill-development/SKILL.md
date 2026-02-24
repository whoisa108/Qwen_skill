---
name: skill-development
description: Guide the creation, improvement, and organization of Claude Code skills (SKILL.md files). Trigger this skill when the user wants to "create a skill", "add a skill", "write a new skill", "improve skill description", "organize skill content", "build a plugin skill", or needs guidance on skill structure, progressive disclosure, or skill development best practices. Also trigger when the user shows you an existing SKILL.md and asks for feedback, or when they want to convert a workflow or prompt into a reusable skill.
---

# Skill Development

Design and write high-quality Claude Code skills. Every decision should serve one goal: **the skill triggers when it should, and works correctly when it does**.

---

## Core Mental Model

A skill has two jobs:
1. **Be found** — the `description` must trigger at the right time
2. **Work correctly** — the body must produce the right output

Most skill failures are either undertriggering (description too narrow) or incorrect execution (body too vague). Address both explicitly.

---

## Skill Anatomy

```
skill-name/
├── SKILL.md          ← required; name + description + instructions
└── references/       ← optional; loaded on demand
    └── *.md
└── scripts/          ← optional; executable helpers
    └── *.sh / *.py
```

**SKILL.md structure:**
```markdown
---
name: skill-name
description: [triggering description — see below]
---

# Skill Title

[Instructions for Claude to follow when this skill activates]
```

---

## Writing the Description (Most Important Part)

The description is the **only thing Claude reads before deciding whether to activate the skill**. It must answer:
- What does this skill do?
- When exactly should it trigger?
- What keywords or phrases should match?

**Pattern that works well:**
```
[What it does]. Trigger this when the user [specific actions/phrases], 
mentions [keywords], or asks about [related concepts]. Also trigger when [edge cases].
```

**Make it slightly pushy** — Claude undertriggers by default. Include:
- Explicit "trigger when..." phrasing
- Synonyms and alternate phrasings
- Non-obvious trigger cases ("also trigger when...")

**Bad description:**
> "Helps with Git operations."

**Good description:**
> "Analyze branch differences and generate Pull Request descriptions. Trigger when user wants to write a PR, open a pull request, summarize branch changes, or mentions 'PR template', 'pull request body', or 'prepare for review'."

---

## Writing the Body

### Progressive Disclosure Pattern

Structure instructions from coarse to fine:

1. **Decision gate** — should this skill run? (if applicable)
2. **Gather evidence first** — always read before acting (QWEN principle)
3. **Step-by-step execution** — numbered, specific, with exact commands
4. **Edge cases** — concrete, not generic
5. **Output format** — what should the final result look like?

### QWEN Alignment

Apply these principles in skill body design:

| QWEN Principle | Skill Implementation |
|---|---|
| Evidence-Based | Start with read/inspect commands before any action |
| `file:line` references | Instruct Claude to cite specific locations |
| Actionable | Use numbered steps, not prose descriptions |
| Incremental | Break execution into verify-before-proceed stages |
| Simple over complex | Prefer explicit commands over abstract heuristics |
| Learn from existing | Include step to read project conventions first |

### Command Examples Are Mandatory

Never say "run the appropriate git command". Say:
```bash
git diff <target>...<current> --stat
```

Claude follows examples literally — vague instructions produce inconsistent results.

---

## Skill Sizing Guide

| Skill size | When to use | Structure |
|---|---|---|
| Single SKILL.md < 200 lines | Focused, single-domain skill | All in SKILL.md |
| SKILL.md + references/ | Multi-variant skill (e.g., AWS vs GCP) | SKILL.md selects; references/ has detail |
| SKILL.md + scripts/ | Deterministic, repeatable operations | Script handles execution; SKILL.md handles reasoning |

Keep SKILL.md under 500 lines. If approaching the limit, extract details to `references/` and add a pointer.

---

## Checklist Before Finalizing

**Description:**
- [ ] Includes explicit "trigger when..." phrasing
- [ ] Covers synonyms and alternate user phrasings
- [ ] Includes at least one non-obvious trigger case
- [ ] Distinguishes from adjacent skills (what it does NOT handle)

**Body:**
- [ ] Starts with information gathering, not action
- [ ] Every command is concrete and copyable
- [ ] Edge cases are listed with specific handling (not "handle appropriately")
- [ ] Output format is described explicitly
- [ ] Uses `file:line` citation pattern where referencing code

---

## Common Mistakes

**Overlap between skills:** Two skills with similar descriptions will confuse the trigger. Add explicit "NOT for X" clauses to each to disambiguate.

**Body that reads like documentation:** Skills should be instructions TO Claude, not explanations FOR the user. Write imperatively: "Read the diff. Group files by concern. Propose before executing."

**Missing confirmation gates:** For any destructive or non-reversible action, require explicit user confirmation step. Document it in the body.

**Vague edge cases:** "Handle errors gracefully" is not an edge case. Write: "If branch not found, report `branch <name> does not exist` and stop."

---

## Workflow for Creating a New Skill

1. **Define scope** — one sentence: "This skill does X when Y"
2. **Write description** — trigger-focused, slightly pushy
3. **Draft body** — gather evidence → decide → execute → verify
4. **Add edge cases** — think: what breaks? what's ambiguous?
5. **Review overlap** — does this conflict with any existing skill?
6. **Test mentally** — would this trigger on the right prompts? Would it miss any?

Ask the user to confirm the trigger description before writing the full body — it's faster to fix the description than to rewrite the body.

---

## References

Read **`reference/skill-development.md`** when:

- **§1 Description Optimization** — writing or improving a skill description, user asks "why isn't my skill triggering?", or you need disambiguation clause patterns, pushiness calibration table, or the mental test framework for validating trigger coverage.
- **§2 Body Writing Patterns** — drafting a skill body for a multi-step workflow, user asks how to structure a complex skill, or you need a specific pattern (Decision Gate, Propose-Confirm-Execute, Grouped Output, Branching Logic, Cleanup+Remind), edge case format, or skill handoff declaration template.
