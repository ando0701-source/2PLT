# 2PLT Audit Checks (Normative)

## 0. Purpose and Scope
This document defines a **mechanizable audit procedure** for evaluating a single 2PLT exchange:
`MANAGER block` → `WORKER response` (and, if produced, the WORKER artifact).

This procedure is intended for:
- an evaluator LLM in an evaluation thread, and/or
- a MANAGER self-check loop.

This document is normative only for **evaluation and self-check**, and does not expand WORKER terminals.

## 1. Inputs (Normative)
An audit run MUST take as input:
1. The exact MANAGER block text.
2. The exact WORKER response text.
3. If the WORKER produced a ZIP artifact: the extracted file tree.

## 2. Verdict Model (Normative)
- **PASS**: all required checks pass.
- **FAIL**: one or more required checks fail.

If FAIL, the audit output SHOULD provide `REQUIRED_TO_RESOLVE` items (see DOC_ID `2PLT_40_OUTPUT_SCHEMA`) that:
- reference the failing `CHECK_ID`,
- identify the minimal `FIX_DOC_ID` set needed to self-repair,
- remain actionable.

## 3. Required Checks (Normative)

### P. Bootstrap Handshake Checks (Non-Activated)

These checks apply only when the evaluated exchange is a **BOOTSTRAP handshake** (DOC_ID `2PLT_10_STATE_MACHINE` §3.1).
If the exchange is an activated MANAGER block, mark these checks as N/A.

| CHECK_ID | Requirement | Source DOC_ID |
|---|---|---|
| P01 | BOOTSTRAP handshake prelude MUST match the exact template and contain only: `BOOTSTRAP_HANDSHAKE`, `BOOTSTRAP_TARGET: WORKER`, `BOOTSTRAP_CONTEXT: CLEAN`, `BOOTSTRAP_ZIP: <filename>`, `BOOTSTRAP_OUTPUT: ACK_ONLY`. | `2PLT_00_DOCUMENT_GOVERNANCE`, `2PLT_10_STATE_MACHINE` |
| P02 | In BOOTSTRAP_HANDSHAKE_MODE, WORKER output MUST be exactly one token in {ACK, NACK, INPUT_MISSING} and MUST be the entire output. | `2PLT_10_STATE_MACHINE` |


### A. MANAGER Block Checks

| CHECK_ID | Requirement | Source DOC_ID |
|---|---|---|
| A01 | MANAGER block MUST conform to the grammar and mandatory fields (OWNER_ID, LANE_ID, REQUEST_ID, trigger token, profile doc id, task section as required by profile). | `2PLT_20_MANAGER_BLOCK_GRAMMAR`, `2PLT_50_PROFILE_*` |
| A02 | MANAGER block MUST use **canonical trigger tokens only**. (Aliases MAY be used by humans, but MUST NOT be emitted by MANAGER.) | `2PLT_00_DOCUMENT_GOVERNANCE`, `2PLT_20_TRIGGER_ID_VOCAB` |

### B. WORKER Response Envelope Checks

| CHECK_ID | Requirement | Source DOC_ID |
|---|---|---|
| B01 | WORKER response MUST include `STATE` and `ARTIFACT`. | `2PLT_40_OUTPUT_SCHEMA` |
| B02 | WORKER response MUST include all Artifact Metadata Fields: `TRIGGER`, `OWNER_ID`, `LANE_ID`, `REQUEST_ID`, `IN_STATE`, `OUT_STATE`, `ARTIFACT_CLASS`, `ARTIFACT_FORMAT`. | `2PLT_20_ARTIFACT_META_VOCAB`, `2PLT_40_OUTPUT_SCHEMA` |
| B03 | Artifact Metadata Fields MUST match the MANAGER block for: `TRIGGER`, `OWNER_ID`, `LANE_ID`, `REQUEST_ID`, and the computed `IN_STATE`. | `2PLT_40_OUTPUT_SCHEMA`, `2PLT_10_STATE_MACHINE` |
| B04 | `OUT_STATE` MUST be consistent with the allowed state transition set and the trigger-terminal matrix. | `2PLT_10_STATE_MACHINE`, `2PLT_30_TRIGGER_TERMINAL_MATRIX` |

### C. Terminal / Artifact Checks

| CHECK_ID | Requirement | Source DOC_ID |
|---|---|---|
| C00 | If `STATE=PROPOSAL`, `ARTIFACT` MUST be `INLINE` and the response MUST include a mechanically-applicable unified diff inline (e.g., `PROPOSED_DIFF:`). | `2PLT_40_OUTPUT_SCHEMA`, `2PLT_50_PROFILE_*` |
| C00b | If `STATE=PROPOSAL`, WORKER response MUST NOT contain `BEGIN_WORKER`/`END_WORKER` wrappers, `OUTPUT_ZIP`, `APPLIED_PATCH`, `RESULT:`, `SHA256:`, or any `sandbox:/` link text. | `2PLT_40_OUTPUT_SCHEMA` |
| C00c | If `STATE=PROPOSAL`, WORKER response MUST NOT claim that a patch was applied (no mutation claims). | `2PLT_40_OUTPUT_SCHEMA`, `2PLT_10_RESPONSIBILITY` |
| C00d | WORKER response MUST NOT contain any `BEGIN_*` / `END_*` wrapper tokens. | `2PLT_40_OUTPUT_SCHEMA` |
| C00e | If `STATE=PROPOSAL`, `NOTES` list items MUST use `- ` as list marker; `* ` bullets are forbidden; and `PROPOSED_DIFF:` MUST be a top-level key (not nested under NOTES). | `2PLT_40_OUTPUT_SCHEMA`, `2PLT_20_OUTPUT_TEMPLATE_VOCAB` |
| C01 | If `STATE=COMMIT`, `ARTIFACT` MUST reference a produced ZIP artifact (filename/path), and the ZIP MUST contain updated files only under `owners/<OWNER_ID>/lanes/<LANE_ID>/`. | `2PLT_10_RESPONSIBILITY`, `2PLT_40_EXECUTION_POLICY`, `2PLT_50_PROFILE_*` |
| C02 | If `STATE=UNRESOLVED`, `ARTIFACT` MUST reference a produced ZIP artifact (filename/path), and the ZIP MUST contain physical logs only under `owners/<OWNER_ID>/lanes/<LANE_ID>/`. | `2PLT_10_RESPONSIBILITY`, `2PLT_40_EXECUTION_POLICY`, `2PLT_50_PROFILE_*` |
| C03 | If `STATE=ABEND`, `ARTIFACT` MUST be INLINE and MUST NOT claim physical file mutation. | `2PLT_40_EXECUTION_POLICY`, `2PLT_40_OUTPUT_SCHEMA` |
| C03b | WORKER response MUST NOT include markdown links such as `[Download ...](...)`. | `2PLT_40_OUTPUT_SCHEMA` |
| C04 | For `IN_STATE=PROPOSAL` with trigger `2PLT_JL_REJECT`, `STATE` MUST be `UNRESOLVED` (ZIP) and MUST persist the reject log update in write-scope. | `2PLT_30_TRIGGER_TERMINAL_MATRIX`, `2PLT_50_PROFILE_JUDGEMENT_LOG_REJECT` |

### D. Failure Reason / Recovery Checks

| CHECK_ID | Requirement | Source DOC_ID |
|---|---|---|
| D01 | If `STATE=UNRESOLVED`, `REASON_CODE` MUST be present and MUST have `RECOVERY_CLASS=RECOVERABLE`. | `2PLT_20_REASON_CODE_VOCAB`, `2PLT_40_OUTPUT_SCHEMA` |
| D02 | If `STATE=UNRESOLVED`, `REQUIRED_TO_RESOLVE` MUST be present and MUST be a **non-empty array of objects** compliant with the output schema. | `2PLT_40_OUTPUT_SCHEMA` |
| D03 | If `STATE=ABEND`, `REASON_CODE` SHOULD be present. If present, it MUST be a valid `reason_code`. | `2PLT_20_REASON_CODE_VOCAB`, `2PLT_40_OUTPUT_SCHEMA` |

## 4. Protocol Invariants to Evaluate (Normative)
The evaluator MUST evaluate the protocol invariants defined in DOC_ID `2PLT_00_MODEL` (section: Protocol Invariants).
At minimum, the audit report MUST include a PASS/FAIL for each invariant that is applicable to the exchange.

## 5. Recommended Evaluation Report Template (Non-normative)

EVALUATION_REPORT
- CONTEXT
  - OWNER_ID: <...>
  - LANE_ID: <...>
  - REQUEST_ID: <...>
  - TRIGGER: <...>
  - PROFILE_DOC_ID: <...>
  - IN_STATE: <...>
- OBSERVED
  - STATE: <...>
  - ARTIFACT: <INLINE|ZIP>
  - OUT_STATE: <...>
- CHECKLIST_RESULTS
  - P01: PASS|FAIL|N/A / note: <...>
  - P02: PASS|FAIL|N/A / note: <...>
  - A01: PASS|FAIL  / note: <...>
  - A02: PASS|FAIL  / note: <...>
  - B01: PASS|FAIL  / note: <...>
  - B02: PASS|FAIL  / note: <...>
  - B03: PASS|FAIL  / note: <...>
  - B04: PASS|FAIL  / note: <...>
  - C01: PASS|FAIL|N/A / note: <...>
  - C02: PASS|FAIL|N/A / note: <...>
  - C03: PASS|FAIL|N/A / note: <...>
  - C04: PASS|FAIL|N/A / note: <...>
  - D01: PASS|FAIL|N/A / note: <...>
  - D02: PASS|FAIL|N/A / note: <...>
  - D03: PASS|FAIL|N/A / note: <...>
- INVARIANTS
  - INV-01: PASS|FAIL / note: <...>
  - INV-02: PASS|FAIL / note: <...>
  - ...
- VERDICT: PASS|FAIL
- REQUIRED_TO_RESOLVE (if FAIL)
  - - CHECK_ID: <...>
      FIX_DOC_ID: [<doc_id>, ...]
      SUMMARY: <one line>
      ACTION: <actionable instruction>

## BOOT LOADER checks (Normative)

- Verify that exactly one BOOT LOADER JSON is selected by trigger.
- Verify that all physical paths listed in BOOT LOADER JSON load order exist.
- Verify that activated processing does not consult documents outside the BOOT-loaded set.
