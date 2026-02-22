# 2PLT_PROFILE_JUDGEMENT_LOG_REJECT (Normative; Profile)

## Purpose
Profile specialization for Judgement Log **REJECT** flow.

This profile exists to make "Manager rejected the proposal" a **machine-auditable terminal record**
without adding a new external terminal outcome.

## Trigger Allowlist
- JL_REJECT (see DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`)

## DocSet Declaration (Normative)

- DOC_SET_ID: MINSET_JL_REJECT_V1
- REQUIRES_DOC_ID:
  - 2PLT_00_MODEL
  - 2PLT_00_ENTRYPOINT
  - 2PLT_10_STATE_MACHINE
  - 2PLT_10_RESPONSIBILITY
  - 2PLT_20_MANAGER_BLOCK_GRAMMAR
  - 2PLT_20_REQUEST_ID_VOCAB
  - 2PLT_20_OWNER_ID_VOCAB
  - 2PLT_20_ARTIFACT_META_VOCAB
  - 2PLT_05_DOC_ID_VOCAB
  - 2PLT_20_TRIGGER_ID_VOCAB
  - 2PLT_30_TRIGGER_TERMINAL_MATRIX
  - 2PLT_40_EXECUTION_POLICY
  - 2PLT_40_OUTPUT_SCHEMA
  - 2PLT_20_REASON_CODE_VOCAB
  - 2PLT_50_PROFILE_JUDGEMENT_LOG_REJECT
  - 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL



## Input Contract

The MANAGER payload MUST include:

- `DOC_SET: MINSET_JL_REJECT_V1` (OPTIONAL label; if present SHOULD match `DOC_SET_ID`)

If missing, WORKER MUST ABEND (REASON_CODE=`SCHEMA_MISSING_REQUIRED`) and include REQUIRED_TO_RESOLVE.

## Terminal Constraint (Normative)
- Preferred terminal: **UNRESOLVED**
- Allowed terminals: UNRESOLVED, ABEND (see DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX`)
- PROPOSAL and COMMIT are prohibited for JL_REJECT.


## Output Envelope Requirements (Normative)

When terminating with UNRESOLVED under JL_REJECT, the WORKER MUST emit:

- STATE=UNRESOLVED
- ARTIFACT=<ZIP artifact id> (the produced artifact is the ZIP described in this profile)
- REASON_CODE: REQUIRED
  - MANAGER_REJECTED_PROPOSAL (normal reject case), or
  - REJECT_TARGET_NOT_FOUND (if no target JL_PROPOSAL can be resolved)
- REQUIRED_TO_RESOLVE: REQUIRED (non-empty array; DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` §10.2)

(Informative) Minimal REQUIRED_TO_RESOLVE example (normal reject):

REQUIRED_TO_RESOLVE:
- CHECK_ID: JL_REJECT terminal and artifacts
  FAIL_REASON_CODE: MANAGER_REJECTED_PROPOSAL
  FIX_KIND: INPUT_REPAIR
  FIX_DOC_ID: 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL
  FIX_SECTION: Acceptable Payload Forms
  FIX_HINT: "Issue a new JL_PROPOSAL in the same (OWNER_ID, LANE_ID) with updated diff/patch, using canonical trigger token."
## Physical Responsibility (Required)
JL_REJECT MUST perform a physical mutation and MUST produce a ZIP artifact (DOC_ID `2PLT_40_OUTPUT_SCHEMA`):

- Write-scope MUST be limited to:
  - `owners/<OWNER_ID>/lanes/<LANE_ID>/`
- The ZIP artifact MUST contain the updated files under that scope.

## Target Binding (Required)
JL_REJECT MUST bind to the most recent JL_PROPOSAL in the same `(OWNER_ID, LANE_ID)` flow.

If no target JL_PROPOSAL can be resolved, JL_REJECT MUST return:
- OUT_STATE=UNRESOLVED with reason_code `REJECT_TARGET_NOT_FOUND`, if a schema-compliant UNRESOLVED ZIP can be produced
- otherwise OUT_STATE=ABEND

## Unresolved Log Record (Required)
JL_REJECT MUST append exactly one JSONL record to:

- `owners/<OWNER_ID>/lanes/<LANE_ID>/log_jl_unresolved_rsp.jsonl`

The record MUST include at least:

- `event`: "MANAGER_REJECT"
- `owner_id`: `<OWNER_ID>`
- `lane_id`: `<LANE_ID>`
- `request_id`: `<REQUEST_ID>`
- `target_request_id`: `<REQUEST_ID of the bound JL_PROPOSAL>`
- `reason_code`: `MANAGER_REJECTED_PROPOSAL` (or `REJECT_TARGET_NOT_FOUND` if target missing)

A matching request-side record MAY be appended to `log_jl_unresolved_req.jsonl` (optional).

