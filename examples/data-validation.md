---
id: data-validation
trigger: "Before processing any user-supplied or externally-sourced data record"
---

## Inputs

- **record** — the data object being validated
- **required_fields** — list of field names that must be present and non-empty
- **schema** — expected types/formats for each field (optional, if defined)

## Tree

START → check_presence

check_presence:
  any required_field is missing → EXIT: fail_missing_fields
  any required_field is empty string → EXIT: fail_empty_fields
  default → check_types

check_types:
  schema is not defined → check_business_rules
  any field fails type check → EXIT: fail_type_mismatch
  default → check_business_rules

check_business_rules:
  record contains obviously invalid value (negative price, future birthdate, etc.) → EXIT: fail_business_rule
  record is a duplicate of an existing record → EXIT: fail_duplicate
  default → EXIT: pass_valid

## Exits

**pass_valid**
- Action: Mark record as VALID, proceed with processing
- Report: "Record validated successfully"
- Next: Return to skill, continue to next processing step

**fail_missing_fields**
- Action: Mark record as INVALID, collect list of missing fields
- Report: "Validation failed — missing required fields: {missing_field_names}"
- Next: Return to skill; skip record or surface to user depending on skill config

**fail_empty_fields**
- Action: Mark record as INVALID, collect list of empty fields
- Report: "Validation failed — empty required fields: {empty_field_names}"
- Next: Return to skill; skip record or surface to user depending on skill config

**fail_type_mismatch**
- Action: Mark record as INVALID, note which fields failed and expected vs actual types
- Report: "Validation failed — type mismatch: {field_name} expected {expected_type}, got {actual_type}"
- Next: Return to skill; skip record or surface to user depending on skill config

**fail_business_rule**
- Action: Mark record as INVALID, note which rule was violated
- Report: "Validation failed — business rule violation: {rule_description}"
- Next: Return to skill; skip record or surface to user depending on skill config

**fail_duplicate**
- Action: Mark record as DUPLICATE, do not process
- Report: "Duplicate record detected — matches existing record ID {existing_id}"
- Next: Return to skill; skip silently or log depending on skill config
