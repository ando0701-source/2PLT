# 2PLT_TRIGGER_TERMINAL_MATRIX

## Purpose
Normative annex to 2PLT_MODEL.

This document fixes:
- trigger_type → allowed terminal outcome set
- prohibited terminal outcomes (hard ban)

Trigger identity and token vocabulary are defined in:
- 2PLT_20_TRIGGER_ID_VOCAB

## Matrix

### Trigger-specific overrides (precedence)

If a `trigger_id` is listed in this section, its allowed/prohibited terminal set overrides the default `trigger_type` mapping below.

| trigger_id | allowed_terminals | prohibited_terminals | notes |
|---|---|---|---|
| JL_REJECT | UNRESOLVED, ABEND | PROPOSAL, COMMIT | Manager rejects the most recent JL_PROPOSAL and records the rejection via UNRESOLVED. |


### PROPOSAL-type triggers (trigger_type=PROPOSAL)
Allowed terminals:
- PROPOSAL
- ABEND

Prohibited terminals:
- COMMIT
- UNRESOLVED

### COMMIT-type triggers (trigger_type=COMMIT)
Allowed terminals:
- COMMIT
- UNRESOLVED
- ABEND

Prohibited terminals:
- PROPOSAL


## Trigger Token Notes (Informative)

- Valid trigger tokens (canonical and aliases) are defined by DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.
- WORKER-side parsing MAY accept aliases for compatibility, but **MANAGER MUST emit canonical tokens only**
  per DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` (section 11).

Canonical examples:
- `@@@@2PLT_JL_PROPOSAL@@@@` → trigger_id `JL_PROPOSAL` → trigger_type `PROPOSAL`
- `@@@@2PLT_JL_COMMIT@@@@` → trigger_id `JL_COMMIT` → trigger_type `COMMIT`