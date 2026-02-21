# 2PLT_OUTPUT_SCHEMA (Normative)

## 1. Purpose

This document defines the normative output contract for a 2PLT terminal response.

## 2. Output Envelope (Required)

A terminal response MUST contain the following keys at top-level (machine-readable):

- STATE: one of {PROPOSAL, COMMIT, UNRESOLVED, ABEND}
- ARTIFACT: one of:
  - INLINE (primary artifact payload is fully contained inline in the response envelope)
  - <relative_artifact_id> (a physically produced/updated artifact; typically a ZIP)


Reserved values:
- The literal string `ZIP` is NOT a valid <relative_artifact_id>; it is only a format label. Use a filename/path (e.g., `TEST-0002_worker_primary_JL_A_COMMIT.zip`).

Note: The response envelope itself is always returned inline; ARTIFACT indicates the primary artifact channel and any required external artifact reference.

Optional keys (terminal-dependent):
- REASON_CODE

## 2.1 Artifact Channel Invariants (Normative)

Unless an active profile explicitly overrides:

- If `STATE=PROPOSAL`  → `ARTIFACT` MUST be `INLINE`.
- If `STATE=ABEND`     → `ARTIFACT` MUST be `INLINE`.
- If `STATE=COMMIT`    → `ARTIFACT` MUST reference a produced ZIP artifact id (filename/path).
- If `STATE=UNRESOLVED`→ `ARTIFACT` MUST reference a produced ZIP artifact id (filename/path).

Additionally:
- A PROPOSAL MUST NOT claim that a patch was applied.
- A PROPOSAL MUST NOT produce any ZIP artifact.

## 2.2 Output Text Restrictions (Normative)

## 2.3 Key-Value Formatting Rules (Normative)

To enable mechanical verification, the WORKER response MUST use the following formatting:

- Each required top-level key MUST be one line in the exact form: `KEY: <value>`.
- Keys MUST NOT be prefixed by list markers (`-` or `*`) and MUST start at column 1.
- Markdown bullets are forbidden (e.g., `* item`). Only `- item` is permitted for list items.

`NOTES:` formatting:
- `NOTES:` MUST be followed immediately by one or more list items.
- Each list item MUST start with `- ` (hyphen + space).
- Blank lines inside `NOTES` are forbidden.

`PROPOSED_DIFF:` formatting:
- `PROPOSED_DIFF:` MUST be a top-level key (not nested under `NOTES`).
- The unified diff block MUST start on the next line and MUST NOT be indented.
- The diff block MUST continue until end-of-output.

Forbidden tokens in all WORKER responses:
- Any `BEGIN_*` / `END_*` wrapper tokens (e.g., `BEGIN_MANAGER`, `END_MANAGER`, `BEGIN_WORKER`, `END_WORKER`).

## 2.4 Output Template Usage (Normative)

If the MANAGER payload contains `OUTPUT_TEMPLATE_ID: <id>` then the WORKER MUST follow the exact template bound to that id
in DOC_ID `2PLT_20_OUTPUT_TEMPLATE_VOCAB`. No extra lines are permitted.


The response envelope MUST be plain text.

Forbidden in all terminals:
- Markdown link syntax (e.g., `[...](...)`).
- WORKER wrapper blocks such as `BEGIN_WORKER` / `END_WORKER`.

Additional forbidden content when `STATE=PROPOSAL`:
- Any claim of application, including tokens like `APPLIED_PATCH`, `RESULT: SUCCESS`, or equivalent.
- Any produced artifact reference, including keys like `OUTPUT_ZIP:` or `SHA256:` for a produced ZIP.
- Any sandbox download/link text (e.g., `sandbox:/...`).

Permitted freeform sections (optional, terminal-dependent) MUST still remain plain text and must not violate the above:
- REQUIRED_TO_RESOLVE
- NOTES
- EVIDENCE
- SUMMARY

## 3. ARTIFACT Semantics

- The terminal response envelope (STATE, ARTIFACT, and optional keys) is always returned inline.
- ARTIFACT=INLINE means: the primary artifact is fully contained inline in the response envelope and does not require any external file to be produced.
- A relative artifact identifier means: a physically produced/updated artifact exists (typically a ZIP). The identifier MUST be a relative path or filename that unambiguously identifies the produced artifact.

Common pattern:
- COMMIT / UNRESOLVED: ARTIFACT identifies a ZIP artifact, while the response envelope still includes inline metadata for auditing.
- PROPOSAL / ABEND: ARTIFACT=INLINE.


## 4. Terminal-Specific Obligations

### PROPOSAL
- STATE=PROPOSAL
- ARTIFACT=INLINE

Additional PROPOSAL strictness (Normative):
- A PROPOSAL response MUST include a mechanically-applicable unified diff inline (recommended key: `PROPOSED_DIFF:`).
- A PROPOSAL response MUST NOT mention any produced ZIP filename; the only ZIP filename that may appear is the MANAGER-supplied `proposal_input_zip` echoed for audit.

- REASON_CODE: MUST NOT be required.
- NOTES MUST include the active profile's mechanically-applicable proposal payload required to enable a later COMMIT
  (e.g., required artifact references and patch payload), unless a profile explicitly defines an equivalent mechanism.

### COMMIT
- STATE=COMMIT
- ARTIFACT: MUST identify the produced/updated artifact (e.g., a ZIP file)
- REASON_CODE: optional (only if policy/profile requires)

### UNRESOLVED
- STATE=UNRESOLVED
- ARTIFACT: MUST identify the produced/updated artifact (typically a ZIP)
- REASON_CODE: REQUIRED (must be a valid reason code)
- REQUIRED_TO_RESOLVE: REQUIRED (non-empty array of REQUIRED_TO_RESOLVE record objects; see schema below and DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` §10.2)
- The produced artifact MUST include the profile-defined unresolved journal/log updates when such a profile is active.

### ABEND
- STATE=ABEND
- ARTIFACT=INLINE
- REASON_CODE: REQUIRED (must be a valid reason code)
- REQUIRED_TO_RESOLVE: REQUIRED when REASON_CODE has RECOVERY_CLASS=RECOVERABLE (see DOC_ID `2PLT_20_REASON_CODE_VOCAB`).
- NOTES: OPTIONAL (SHOULD include missing items when applicable)




## 5. Reason Code Validity

REASON_CODE (and REASON_CODES if present) MUST be members of DOC_ID `2PLT_20_REASON_CODE_VOCAB`.

Rules:
- STATE is mandatory and singular.
- REASON_CODE MUST NOT duplicate any element in REASON_CODES.
---


## 6. REQUIRED_TO_RESOLVE Schema (Normative)

When present, REQUIRED_TO_RESOLVE MUST be a **non-empty array** of objects.
Each object MUST contain the required keys defined in DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` §10.2:

- CHECK_ID
- FAIL_REASON_CODE
- FIX_KIND
- FIX_DOC_ID
- FIX_SECTION
- FIX_HINT

Constraints:
- FAIL_REASON_CODE MUST be a valid `reason_code` (DOC_ID `2PLT_20_REASON_CODE_VOCAB`).
- FIX_DOC_ID MUST be a valid DOC_ID (DOC_ID `2PLT_05_DOC_ID_VOCAB`) and MUST NOT be a physical filename.
- FIX_HINT MUST be finite and mechanically actionable (no ambiguous instructions).

(Informative) Minimal example:

REQUIRED_TO_RESOLVE:
- CHECK_ID: JL_REJECT terminal and artifacts
  FAIL_REASON_CODE: MANAGER_REJECTED_PROPOSAL
  FIX_KIND: INPUT_REPAIR
  FIX_DOC_ID: 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL
  FIX_SECTION: Acceptable Payload Forms
  FIX_HINT: "Issue a new JL_PROPOSAL in the same (OWNER_ID, LANE_ID) with input_zip + patch_target + diff/patch, using canonical trigger token."
## Artifact Metadata Fields (Normative)

The WORKER output MUST include the following fields, defined by DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`:

- TRIGGER
- OWNER_ID
- LANE_ID
- REQUEST_ID
- IN_STATE
- OUT_STATE
- ARTIFACT_CLASS
- ARTIFACT_FORMAT


Constraints:

- OUT_STATE MUST equal STATE.
- TRIGGER MUST be the canonical trigger token as defined for the resolved trigger_id in DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.
- OWNER_ID MUST equal the MANAGER block `OWNER_ID` value parsed per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`.
- LANE_ID MUST equal the MANAGER block `LANE_ID` value parsed per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`.
- REQUEST_ID MUST equal the MANAGER block `REQUEST_ID` value parsed per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`.
- If the MANAGER block used an alias token, WORKER MAY echo it in NOTES for audit, but TRIGGER remains canonical.
- If ARTIFACT identifies a ZIP artifact then ARTIFACT_FORMAT MUST be ZIP.
- If ARTIFACT=INLINE then ARTIFACT_FORMAT MUST be INLINE.
- The default terminal pairing (STATE -> ARTIFACT_CLASS/FORMAT) MUST follow DOC_ID `2PLT_20_ARTIFACT_META_VOCAB` unless the active profile explicitly overrides it.

