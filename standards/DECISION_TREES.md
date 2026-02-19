# Decision Trees Standard

**Version:** 1.0
**Part of:** OpenClaw conventions

---

## Purpose

This standard defines the format for decision tree files used by OpenClaw agents. Trees encode branching logic that would otherwise be buried in skill prose.

---

## File Naming

- Tree files live in a `trees/` subdirectory within the skill folder
- Named after the decision they handle: `image-assessment.md`, `error-fallback.md`
- Lowercase, hyphen-separated, descriptive

---

## Required Sections

Every tree file must have these four sections in order:

### 1. Header (YAML front matter)

```yaml
id: tree-name
trigger: "When this tree is entered — be specific about the moment"
```

- `id` matches the filename (without `.md`)
- `trigger` is a plain-English description of when to enter this tree

### 2. Inputs

```markdown
## Inputs

- **variable_name** — description of what it contains
- **another_variable** — description
```

List every variable the tree expects to be in scope before it runs. If an input is missing, the agent must halt and surface an error rather than guessing.

### 3. Tree

```markdown
## Tree

START → first_node

first_node:
  condition_a → node_b
  condition_b → node_c
  default → node_d

node_b:
  sub_condition_1 → EXIT: outcome_one
  sub_condition_2 → EXIT: outcome_two

node_c:
  → EXIT: outcome_three

node_d:
  → EXIT: outcome_four
```

**Rules:**
- `START →` always points to the first node
- Each node name is a verb or noun phrase in `snake_case`
- Conditions are plain English, not code
- Terminal nodes use `EXIT: label` — label matches a key in the Exits section
- Use `default →` for the catch-all branch when no other condition matches
- No implicit fallthrough — every branch must be explicit

### 4. Exits

```markdown
## Exits

**outcome_one**
- Action: what the agent does
- Report: what the agent says/logs
- Next: what happens after (return to SKILL.md step N, or trigger another tree)

**outcome_two**
- Action: ...
- Report: ...
- Next: ...
```

Every `EXIT: label` in the Tree section must have a matching entry here.

---

## Formatting Rules

1. **One decision per file.** If a node gets complex enough to need its own sub-tree, extract it into a new file.
2. **No prose in Tree section.** Conditions are labels, not sentences. Put explanations in the Exits section.
3. **All branches explicit.** Never rely on the agent to infer a missing branch.
4. **Consistent indentation.** Two spaces under each node name.
5. **No logic in SKILL.md.** The skill file references the tree by name. It does not duplicate or summarize the tree's logic.

---

## Example (minimal)

```markdown
---
id: retry-check
trigger: "After an API call fails"
---

## Inputs

- **attempt_count** — how many times this call has been tried
- **error_code** — HTTP status code returned

## Tree

START → evaluate_error

evaluate_error:
  error_code == 429 → rate_limited
  error_code >= 500 → server_error
  error_code >= 400 → client_error

rate_limited:
  attempt_count < 3 → EXIT: retry_with_backoff
  attempt_count >= 3 → EXIT: abort_rate_limit

server_error:
  attempt_count < 2 → EXIT: retry_immediate
  attempt_count >= 2 → EXIT: abort_server_error

client_error:
  → EXIT: abort_client_error

## Exits

**retry_with_backoff**
- Action: Wait 30s × attempt_count, then retry
- Report: "Rate limited. Retrying in Xs (attempt N)"
- Next: Re-enter this tree after retry

**abort_rate_limit**
- Action: Stop execution
- Report: "Rate limit exceeded after 3 attempts. Manual intervention required."
- Next: Surface to user

**retry_immediate**
- Action: Retry immediately
- Report: "Server error (5xx). Retrying (attempt N)"
- Next: Re-enter this tree after retry

**abort_server_error**
- Action: Stop execution
- Report: "Server error persists after 2 attempts."
- Next: Surface to user

**abort_client_error**
- Action: Stop execution
- Report: "Client error (error_code). Check request parameters."
- Next: Surface to user
```

---

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| Prose conditions ("if the image looks bad") | Agent interprets subjectively | Use observable criteria ("brightness < threshold", "contains_text == true") |
| Missing default branch | Agent gets stuck on unexpected input | Always add `default →` |
| Logic duplicated in SKILL.md | Skill and tree drift out of sync | SKILL.md only references, never describes |
| One giant tree with 20+ nodes | Hard to follow, hard to update | Split into multiple focused trees |
| Exits without Next | Agent doesn't know what to do after | Every exit must specify what comes next |
