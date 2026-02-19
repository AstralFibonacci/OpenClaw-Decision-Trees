# OpenClaw Decision Trees

**Structured decision logic for autonomous AI agents.**

Decision trees separate branching logic from execution procedures in your OpenClaw skills. Instead of burying `if/else` logic in prose that your agent has to parse, you extract it into standalone tree files that the agent follows mechanically.

**The problem:** Agent skills written in markdown embed decision logic as prose, tables, and inline flowcharts. The agent re-reads paragraphs to figure out what to do next, misses edge cases buried in "however, if X then Y" sentences, and makes inconsistent decisions on repeated runs.

**The fix:** Pull decision logic into dedicated `.md` files with a consistent structure. The skill file handles *execution* (API calls, steps). The tree files handle *decisions* (what to do and when).

---

## Before & After

### Before (inline logic in SKILL.md)
```markdown
#### Image Assessment
Check each image. If it's clean with no text, keep it. If it has good
photography but text overlays, flag it for cleaning. However, if the image
shows the wrong variant or a bad angle, keep it temporarily as reference
but mark for deletion later. Blurry or duplicate images should be deleted.

After categorizing, if most images are clean, minimal work needed. If most
have text overlays, prioritize cleaning. If most are unusable, generate new ones.
```

### After (tree reference in SKILL.md)
```markdown
#### Image Assessment
**Decision:** Follow `trees/image-assessment.md`

Outputs: Each image categorized A/B/C/D, overall strategy determined.
```

And in `trees/image-assessment.md`:
```markdown
## Tree

START → check_each_image

check_each_image:
  clean_photo_no_text → CATEGORY_A (keep as-is)
  good_photo_with_text → CATEGORY_B (flag for cleaning)
  wrong_variant_or_bad_angle → CATEGORY_C (reference only)
  blurry_or_duplicate → CATEGORY_D (mark for deletion)

after_categorization → count_categories

count_categories:
  category_a_count > 50% of total → EXIT: minimal_work
  category_b_count > 50% of total → EXIT: cleaning_strategy
  category_c_plus_d > 50% of total → EXIT: generation_strategy
```

The agent hits the decision point, loads the tree, follows it mechanically, and returns to the skill file. No prose parsing. No missed branches.

---

## Installation

### 1. Add the standard to your workspace root

Copy `standards/DECISION_TREES.md` to your OpenClaw workspace:

```bash
cp standards/DECISION_TREES.md ~/your-workspace/standards/
```

### 2. Add the convention to your AGENTS.md

Paste the snippet from `agents-snippet.md` into your `AGENTS.md` file.

### 3. Create trees for your skills

For any skill with complex branching logic, create a `trees/` subdirectory:

```
your-workspace/
├── AGENTS.md                    ← updated with convention
├── standards/
│   └── DECISION_TREES.md       ← format reference
└── skills/
    └── your-skill/
        ├── SKILL.md             ← references trees, no inline logic
        └── trees/
            ├── some-decision.md
            └── another-decision.md
```

---

## File Structure

```
openclaw-decision-trees/
├── README.md                           ← you are here
├── standards/
│   └── DECISION_TREES.md              ← the format standard (copy to workspace root)
├── agents-snippet.md                   ← copy-paste block for your AGENTS.md
└── examples/
    ├── image-assessment.md            ← example: image categorization tree
    ├── error-fallback.md              ← example: API error handling with retries
    └── data-validation.md             ← example: input validation tree
```

---

## Tree File Format

Every tree file has four sections:

### Header
```yaml
id: image-assessment
trigger: "Phase 2 — when evaluating supplier images"
```

### Inputs
What the agent needs before entering the tree.

### Tree
Structured decision paths with explicit branches and exits.

### Exits
What happens at each terminal node — actions to take and what to report.

See `standards/DECISION_TREES.md` for the full specification.

---

## When to Use Decision Trees

| Scenario | Use a Tree? |
|----------|-------------|
| 3+ branching paths | Yes |
| Fallback/retry logic | Yes |
| Agent previously made wrong decision here | Yes |
| Simple if/else, unlikely to change | Keep inline |
| Linear execution (step 1, step 2, step 3) | Keep in SKILL.md |
| API calls and curl commands | Keep in SKILL.md |

---

## How It Compounds

Every time your agent makes a mistake on a decision, you add a branch to the relevant tree. Over time:

- The agent gets more accurate without you rewriting skill files
- New edge cases get caught mechanically instead of buried in prose
- Skills stay readable (execution steps) while trees handle complexity
- Other team members (or other agents) can follow the same decision paths

---

## License

MIT — use it however you want.
