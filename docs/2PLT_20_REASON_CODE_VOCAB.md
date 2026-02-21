# 2PLT_REASON_CODE_VOCAB (Normative)

## Purpose
Fixed vocabulary for `reason_code` used by UNRESOLVED / ABEND outputs.

Keep this list small and stable. Additions require explicit approval.

## Vocabulary (vNext)
- TRIGGER_INVALID
- SCHEMA_MISSING_REQUIRED
- POLICY_VIOLATION
- POLICY_CONFLICT: two or more MUST obligations conflict and cannot be jointly satisfied
- EXECUTION_IMPOSSIBLE
- ARTIFACT_GENERATION_FAILED
- LOG_PERSISTENCE_FAILED
- MANAGER_REJECTED_PROPOSAL: manager explicitly rejected the most recent PROPOSAL; logged via UNRESOLVED
- REJECT_TARGET_NOT_FOUND: JL_REJECT could not resolve a target JL_PROPOSAL within the same (OWNER_ID, LANE_ID)
- INTERNAL_ERROR

- INPUT_MISSING: required input payload is missing or empty
- OWNER_ID_MISSING: required OWNER_ID directive is missing from the MANAGER block
- OWNER_ID_INVALID: OWNER_ID directive is malformed, duplicated, or violates the owner_id format
- OWNER_ID_RESERVED: OWNER_ID is a reserved identifier (see DOC_ID `2PLT_20_OWNER_ID_VOCAB`)
- LANE_ID_MISSING: required LANE_ID directive is missing from the MANAGER block
- LANE_ID_INVALID: LANE_ID directive is malformed, duplicated, or violates the lane_id format
- REQUEST_ID_MISSING: required REQUEST_ID directive is missing from the MANAGER block
- REQUEST_ID_INVALID: REQUEST_ID directive is present but malformed or duplicated
- REQUEST_ID_RESERVED: REQUEST_ID is reserved per DOC_ID 2PLT_20_REQUEST_ID_VOCAB
- WRITE_SCOPE_VIOLATION: a proposed or applied patch would modify files outside the allowed lane-scoped write area
- INPUT_INSUFFICIENT_BUT_PERSISTENCE_REQUIRED: input is insufficient to complete the target action, but log persistence is still required (COMMIT-type triggers)

## Recovery Class (Normative)

Each `reason_code` has a `RECOVERY_CLASS`:

- **RECOVERABLE**: the caller (typically MANAGER) can take an actionable corrective step and re-run.
  - When a RECOVERABLE reason_code is emitted (UNRESOLVED or ABEND), `REQUIRED_TO_RESOLVE` MUST be present.
- **FATAL**: protocol violation or internal/system failure that prevents a reliable actionable plan.
  - `REQUIRED_TO_RESOLVE` is OPTIONAL.
  - If `REQUIRED_TO_RESOLVE` is present, it MUST contain only actionable items.

`RECOVERY_CLASS` table:

| reason_code | RECOVERY_CLASS |
|---|---|
| TRIGGER_INVALID | FATAL |
| POLICY_VIOLATION | FATAL |
| INTERNAL_ERROR | FATAL |
| SCHEMA_MISSING_REQUIRED | RECOVERABLE |
| POLICY_CONFLICT | RECOVERABLE |
| EXECUTION_IMPOSSIBLE | RECOVERABLE |
| ARTIFACT_GENERATION_FAILED | RECOVERABLE |
| LOG_PERSISTENCE_FAILED | RECOVERABLE |
| MANAGER_REJECTED_PROPOSAL | RECOVERABLE |
| REJECT_TARGET_NOT_FOUND | RECOVERABLE |
| INPUT_MISSING | RECOVERABLE |
| OWNER_ID_MISSING | RECOVERABLE |
| OWNER_ID_INVALID | RECOVERABLE |
| OWNER_ID_RESERVED | RECOVERABLE |
| INPUT_INSUFFICIENT_BUT_PERSISTENCE_REQUIRED | RECOVERABLE |
| LANE_ID_MISSING | RECOVERABLE |
| LANE_ID_INVALID | RECOVERABLE |
| REQUEST_ID_MISSING | RECOVERABLE |
| REQUEST_ID_INVALID | RECOVERABLE |
| REQUEST_ID_RESERVED | RECOVERABLE |
| WRITE_SCOPE_VIOLATION | RECOVERABLE |

## Additional Reason Codes (Normative; Phase1)

| reason_code | recovery_class | description |
|---|---|---|
| PROTOCOL_VIOLATION | RECOVERABLE | Output violated mandatory protocol constraints; regenerate the response strictly per output schema and active profile. |
| PROPOSAL_ARTIFACT_VIOLATION | RECOVERABLE | PROPOSAL illegally produced or referenced a ZIP artifact or claimed mutation; regenerate as INLINE diff-only proposal. |
