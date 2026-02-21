# 2PLT_PROFILE_JUDGEMENT_LOG_PROPOSAL (Normative; Profile)

## Purpose
Profile specialization for Judgement Log PROPOSAL flow.

## Trigger Allowlist
- JL_PROPOSAL (see DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`)

## DocSet Declaration (Normative)

- DOC_SET_ID: MINSET_JL_PROPOSAL_V1
- REQUIRES_DOC_ID:
  - 2PLT_00_MODEL
  - 2PLT_20_DOC_SET_VOCAB
  - 2PLT_20_OUTPUT_TEMPLATE_VOCAB
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
  - 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL



## Policy Overrides
- Output MUST follow DOC_ID `2PLT_40_OUTPUT_SCHEMA`.
- Physical mutations are prohibited (see DOC_ID `2PLT_10_RESPONSIBILITY`).

## Artifact
- ARTIFACT=INLINE
- PROPOSAL MUST NOT produce any ZIP artifact.
- PROPOSAL MUST NOT apply any patch; it only proposes a `diff/patch` inline.
- PROPOSAL MUST NOT include markdown download links.

## Output Requirements (Normative)

The WORKER MUST follow output formatting rules in DOC_ID `2PLT_40_OUTPUT_SCHEMA` and MUST use the output template referenced by:
- `OUTPUT_TEMPLATE_ID: JL_PROPOSAL_ENVELOPE_V1` (DOC_ID `2PLT_20_OUTPUT_TEMPLATE_VOCAB`).


A PROPOSAL response MUST include in `NOTES`:

- `doc_set: MINSET_JL_PROPOSAL_V1`
- `proposal_mode: DIFF_ONLY`
- `proposal_input_zip: <zip_filename>` (if supplied by MANAGER)
- `proposal_target: <relative/path>` (one or more)
- `proposed_diff: unified`

This exists to keep TC01-level tests mechanizable and to avoid ambiguous "document hunting" during execution.
Additionally, this profile hard-separates PROPOSAL from COMMIT: PROPOSAL generates *only* a diff; COMMIT applies it.

This exists to keep TC01-level tests mechanizable and to avoid ambiguous "document hunting" during execution.

## Input Contract

The MANAGER block MUST contain:

1. A valid `OWNER_ID: <owner_id>` directive (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`).
2. A valid `LANE_ID: <lane_id>` directive (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`).
3. A trigger token that resolves to trigger_id `JL_PROPOSAL` (canonical only; see DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`).
4. A payload line `DOC_SET: MINSET_JL_PROPOSAL_V1` (see DOC_ID `2PLT_20_DOC_SET_VOCAB`).
5. A payload line `PROPOSAL_MODE: DIFF_ONLY`.
6. A payload line `OUTPUT_TEMPLATE_ID: JL_PROPOSAL_ENVELOPE_V1` (see DOC_ID `2PLT_20_OUTPUT_TEMPLATE_VOCAB`).
7. A payload following the trigger token that provides a **commit-capable proposal specification**.
   - Recommended canonical token: `@@@@2PLT_JL_PROPOSAL@@@@`

This profile treats PROPOSAL as a commit-capable artifact proposal (not a conversational draft).
Therefore, the MANAGER payload MUST provide a deterministic specification (PATCH_INTENT) from which the WORKER can generate a unified diff.

### Required Elements (Commit-Capable Proposal)

A PROPOSAL MUST:

- Identify the `proposal_input_zip` (the authoritative input snapshot) being proposed against (recommended).
- Identify one or more concrete `proposal_target` paths (file path(s) to be changed within the working copy).
- Provide a **PATCH_INTENT** specification that deterministically defines the desired change.

Normative ban: In this profile, the MANAGER payload MUST NOT use the key `patch_target`. That key is reserved for COMMIT-stage payloads. Use `proposal_target` instead. that deterministically defines the desired change.
  - Example forms: "create file with exact content", "append these exact lines", "replace exact line range", etc.

The WORKER MUST generate a mechanically-applicable `diff/patch` (preferred: unified diff) as part of the PROPOSAL response (inline).


### Write Scope (Normative; Owner+Lane Isolation)

For (OWNER_ID, LANE_ID)-scoped operation, a PROPOSAL MUST be **owner+lane-local**:

Allowed write prefix (MUST apply to every patch target):

- `owners/<OWNER_ID>/lanes/<LANE_ID>/`

Normative constraints:

- Every `proposal_target` MUST be under the allowed write prefix.
- The provided `diff/patch` MUST NOT modify any file outside the allowed write prefix for this execution.
- Root-level logs (e.g., `L00_JL_GLOBAL.LOG`, `L00_JL_COMMON.LOG`) are treated as **read-only templates**.
  - Proposals MUST target owner+lane-local copies, e.g.:
    - `owners/<OWNER_ID>/lanes/<LANE_ID>/L00_JL_GLOBAL.LOG`
    - `owners/<OWNER_ID>/lanes/<LANE_ID>/L00_JL_COMMON.LOG`
- If the owner+lane-local target file(s) do not exist in `input_zip`, the proposal MAY include a creation patch that creates them
  (by copying from the root-level template or by initializing empty), but only under the allowed write prefix.

If the MANAGER payload specifies a `proposal_target` outside the allowed lane write scope, the WORKER MUST terminate with ABEND
(REASON_CODE=`WRITE_SCOPE_VIOLATION`) and include REQUIRED_TO_RESOLVE that instructs the MANAGER to regenerate the proposal with owner+lane-local targets.

### Acceptable Payload Forms (Recommended)

The payload MAY be expressed in any readable form, but MUST include the three required elements.
Recommended minimal structure:

- proposal_input_zip: <zip_filename>
- proposal_target:
  - <relative/path/to/file>
- diff: |
  --- a/<relative/path/to/file>
  +++ b/<relative/path/to/file>
  @@ ...

### proposal_input_zip Resolution Rule (Session-Safe)

If `proposal_input_zip` is not explicitly stated in the payload, WORKER MAY infer it ONLY if exactly one
authoritative input ZIP has been acknowledged in the current lane context. If inferred, WORKER MUST
echo the resolved `proposal_input_zip` explicitly in NOTES.

If `proposal_input_zip` cannot be determined under this rule, it is missing.

### Failure Handling

If any required element is missing or not mechanically applicable, the turn MUST terminate with:

- STATE=ABEND
- ARTIFACT=INLINE
- REASON_CODE=INPUT_MISSING

When REASON_CODE has RECOVERY_CLASS=RECOVERABLE, REQUIRED_TO_RESOLVE MUST be present and MUST follow DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` §10.2.

(Informative) Minimal REQUIRED_TO_RESOLVE example for missing payload elements:

REQUIRED_TO_RESOLVE:
- CHECK_ID: INPUT payload completeness
  FAIL_REASON_CODE: INPUT_MISSING
  FIX_KIND: INPUT_REPAIR
  FIX_DOC_ID: 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL
  FIX_SECTION: Acceptable Payload Forms
  FIX_HINT: "Provide PROPOSAL_MODE: DIFF_ONLY, proposal_input_zip (optional but recommended), proposal_target (lane-local relative paths), and a mechanically applicable proposed_diff (unified diff)."

If the payload is missing or empty, the turn MUST terminate with terminal = ABEND and REASON_CODE=INPUT_MISSING.
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



## Optional Lane-Local Unresolved Journal Files (Informative)

If the MANAGER chooses to persist unresolved request/response records as physical files (e.g., inside a ZIP artifact),
the recommended lane-local filenames are:

- `log_jl_unresolved_req.jsonl`
- `log_jl_unresolved_rsp.jsonl`

Recommended placement is under an owner+lane-scoped directory:

- `owners/<OWNER_ID>/lanes/<LANE_ID>/`

This recommendation is informational and does not override DOC_ID `2PLT_10_RESPONSIBILITY` unless a future profile explicitly
permits/defines physical artifacts for UNRESOLVED.