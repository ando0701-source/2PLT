# 2PLT_PROFILE_JUDGEMENT_LOG_COMMIT (Normative; Profile)

## Purpose
Profile specialization for Judgement Log COMMIT flow.

## Trigger Allowlist
- JL_COMMIT (see DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`)

## DocSet Declaration (Normative)

- DOC_SET_ID: MINSET_JL_COMMIT_V1
- REQUIRES_DOC_ID:
  - 2PLT_00_MODEL
  - 2PLT_20_DOC_SET_VOCAB
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
  - 2PLT_50_PROFILE_JUDGEMENT_LOG_COMMIT
  - 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL



## Policy Overrides
- Output MUST follow DOC_ID `2PLT_40_OUTPUT_SCHEMA`.
- Physical mutation is REQUIRED for STATE=COMMIT and MUST produce/update the specified artifact.

## Input Contract

The MANAGER payload MUST include:

- `DOC_SET: MINSET_JL_COMMIT_V1` (see DOC_ID `2PLT_20_DOC_SET_VOCAB`)

If missing, WORKER MUST ABEND (REASON_CODE=`SCHEMA_MISSING_REQUIRED`) and include REQUIRED_TO_RESOLVE.

## Proposal→Commit Linkage (Required)

A JL_COMMIT MUST bind to the most recent JL_PROPOSAL in the same owner (OWNER_ID) and same lane (LANE_ID) within the same session.

If no prior JL_PROPOSAL exists in-session, or the linked proposal lacks a mechanically-applicable patch,
the turn MUST terminate with UNRESOLVED (REASON_CODE=EXECUTION_IMPOSSIBLE) and include REQUIRED_TO_RESOLVE.
If a schema-compliant UNRESOLVED output cannot be produced, the system MUST escalate to ABEND per DOC_ID `2PLT_40_EXECUTION_POLICY`.


## Write Scope (Normative; Owner+Lane Isolation)

To prevent cross-lane and cross-process log mixing, JL_COMMIT is **owner+lane-local**:

- WORKER MUST treat the write scope as the allowed owner+lane prefix:

  - `owners/<OWNER_ID>/lanes/<LANE_ID>/`
- The linked proposal's `patch_target` and `diff/patch` MUST NOT modify any file outside the allowed owner+lane write scope.
- If the patch would modify files outside the lane scope, the turn MUST terminate with UNRESOLVED
  (REASON_CODE=`WRITE_SCOPE_VIOLATION`) and include REQUIRED_TO_RESOLVE per DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` §10.2 (e.g., FIX_HINT instructs to regenerate a lane-local proposal patch).
  - If a schema-compliant UNRESOLVED output cannot be produced, escalate to ABEND per DOC_ID `2PLT_40_EXECUTION_POLICY`.

Lane-local log baselines:

- If lane-local JL log files do not exist yet in the working copy, WORKER SHOULD create them before applying the patch:
  - Prefer copying from root-level templates (`L00_JL_GLOBAL.LOG`, `L00_JL_COMMON.LOG`) into:
    - `owners/<OWNER_ID>/lanes/<LANE_ID>/L00_JL_GLOBAL.LOG`
    - `owners/<OWNER_ID>/lanes/<LANE_ID>/L00_JL_COMMON.LOG`
  - If templates are unavailable, initialize the lane-local files as empty logs.

Copy-through constraint (recommended for auditability):

- Files outside the allowed owner+lane write scope SHOULD remain byte-identical between input_zip and output artifact ZIP (copy-through),
  unless an explicit future profile expands the allowed write scope.

## Commit Behavior (Normative)

Given the linked JL_PROPOSAL:

- WORKER MUST apply the proposal's `diff/patch` to the proposal's `input_zip` working copy.
- WORKER MUST produce a new ZIP artifact containing the updated log(s).
- ARTIFACT in the COMMIT output MUST identify the produced ZIP artifact.
- NOTES SHOULD summarize:
  - input_zip used
  - patch_target(s)
  - whether patch applied cleanly
  - output artifact name

If patch application fails mechanically or the required ZIP artifact cannot be produced,
the turn MUST terminate with UNRESOLVED (REASON_CODE=ARTIFACT_GENERATION_FAILED) and include REQUIRED_TO_RESOLVE.
If a schema-compliant UNRESOLVED output cannot be produced, the system MUST escalate to ABEND per DOC_ID `2PLT_40_EXECUTION_POLICY`.
---

## Artifact Metadata (Normative)

This profile MUST include the artifact metadata fields defined in DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`:

- TRIGGER
- OWNER_ID
- LANE_ID
- IN_STATE
- OUT_STATE
- ARTIFACT_CLASS
- ARTIFACT_FORMAT



## UNRESOLVED Behavior (Normative; JL_COMMIT Failures)

When this profile terminates with STATE=UNRESOLVED (per DOC_ID `2PLT_40_EXECUTION_POLICY`), WORKER MUST:

1) Produce a ZIP artifact (ARTIFACT in the response MUST identify it), and
2) Update lane-local unresolved journal files under the owner+lane write scope:

- Directory: `owners/<OWNER_ID>/lanes/<LANE_ID>/`
- Files (JSON Lines / one JSON object per line):
  - `log_jl_unresolved_req.jsonl`
  - `log_jl_unresolved_rsp.jsonl`

Minimum content requirements:

- `log_jl_unresolved_req.jsonl` MUST append a record containing at least:
  - REQUEST_ID
  - TRIGGER, OWNER_ID, LANE_ID
  - IN_STATE
  - a stable pointer to the input snapshot (e.g., input_zip identifier if known)

- `log_jl_unresolved_rsp.jsonl` MUST append a record containing at least:
  - OUT_STATE=UNRESOLVED
  - REASON_CODE
  - REQUIRED_TO_RESOLVE (non-empty array; see DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` §10.2)
  - TRIGGER, OWNER_ID, LANE_ID
  - IN_STATE

If the lane-local unresolved journal files cannot be updated, or the required ZIP artifact cannot be produced,
the turn MUST escalate to ABEND per DOC_ID `2PLT_40_EXECUTION_POLICY`.

