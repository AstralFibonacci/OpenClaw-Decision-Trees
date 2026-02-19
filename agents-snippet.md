# AGENTS.md Snippet

Copy and paste the block below into your workspace's `AGENTS.md` file.

---

```markdown
## Decision Trees

Skills with branching logic separate decision trees from execution procedures.

### Convention
- Trees live in a `trees/` subdirectory within the skill folder
- Each tree is a standalone `.md` file named after the decision it handles
- SKILL.md references trees inline: "Follow `trees/image-assessment.md`"
- Format defined in `standards/DECISION_TREES.md`

### Execution Rule
When a skill has a `trees/` directory, load ALL tree files during pre-flight
alongside SKILL.md. When you reach a decision point that has a corresponding
tree file, follow the tree logic exactly — the tree overrides any prose
description of the same decision.

### Hard Rule
Decision logic NEVER goes in SKILL.md. If a new decision branch is needed,
it goes into an existing tree file or a new tree file. SKILL.md only references
trees — it never duplicates their logic.

### When to Create Trees
- If a decision has 3+ branches → tree file
- If a decision has fallback/retry logic → tree file
- If the agent has made a wrong decision on something before → tree file
- Simple if/else that's unlikely to change → keep inline in SKILL.md
```
