# 2PLT_20_ARTIFACT_META_VOCAB

## Purpose

This document defines the canonical vocabulary for artifact metadata emitted by the WORKER.
It exists to strengthen STRICT behavior and cross-layer binding between:

- State (2PLT_10_STATE_MACHINE)
- Physical (2PLT_40_EXECUTION_POLICY)
- Profile (2PLT_50_PROFILE_*)

## Fields

### TRIGGER
- Meaning: The trigger token that activated the current execution.
- Source: MANAGER block token (validated via DOC_ID 2PLT_20_TRIGGER_ID_VOCAB).
- Canonical form: The canonical trigger token as recorded in 2PLT_20_TRIGGER_ID_VOCAB.



### LANE_ID
- Meaning: Logical execution lane identifier (session partition key) for this terminal response.
- Source: `LANE_ID: <lane_id>` directive inside the MANAGER block (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`).
- Constraint: LANE_ID MUST exactly equal the parsed MANAGER block lane_id.
- Purpose: Enables safe re-entrancy and prevents proposal→commit linkage and monitoring/aggregation from mixing independent flows.

### OWNER_ID
- Meaning: Logical execution owner identifier (execution route / agent namespace) for this terminal response.
- Source: `OWNER_ID: <owner_id>` directive inside the MANAGER block (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`).
- Constraint: OWNER_ID MUST exactly equal the parsed MANAGER block owner_id.
- Purpose: Prevents cross-owner mixing when multiple WORKER endpoints or agent instances write to a shared storage or when logs are aggregated.


### REQUEST_ID
- Meaning: Request identifier (idempotency key) for this terminal response.
- Source: `REQUEST_ID: <request_id>` directive inside the MANAGER block (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`).
- Constraint: REQUEST_ID MUST exactly equal the parsed MANAGER block request_id.
- Syntax: MUST satisfy DOC_ID `2PLT_20_REQUEST_ID_VOCAB`.
- Purpose: Enables safe retries, correlation, and deduplication by monitoring/audit agents within a session partitioned by (OWNER_ID, LANE_ID).


### IN_STATE
- Meaning: The **source state** of the final transition edge that produced OUT_STATE in this activated turn.
- Allowed values: NUL | PROPOSAL | COMMIT | UNRESOLVED | ABEND
- NUL: Start-of-turn (no predecessor state).

Deterministic mapping (default):
- trigger_type=PROPOSAL:
  - OUT_STATE=PROPOSAL → IN_STATE=NUL
  - OUT_STATE=ABEND    → IN_STATE=PROPOSAL
- trigger_type=COMMIT:
  - OUT_STATE=COMMIT      → IN_STATE=PROPOSAL
  - OUT_STATE=UNRESOLVED  → IN_STATE=COMMIT
  - OUT_STATE=ABEND       →
    - IN_STATE=UNRESOLVED iff UNRESOLVED gate passed (DOC_ID `2PLT_40_EXECUTION_POLICY`, section 4) but a schema-compliant UNRESOLVED output could not be produced.
    - otherwise IN_STATE=COMMIT


Trigger-specific override:
- trigger_id=JL_REJECT:
  - OUT_STATE=UNRESOLVED → IN_STATE=PROPOSAL
  - OUT_STATE=ABEND      → IN_STATE=UNRESOLVED (iff an UNRESOLVED output was required but could not be produced); otherwise IN_STATE=PROPOSAL

- Note: BOOTSTRAP/ACK is a **non-activated prelude** defined by DOC_ID `2PLT_10_STATE_MACHINE` (section 3.1) and MUST NOT be represented via IN_STATE.

### OUT_STATE
- Meaning: The terminal state of the current response.
- Allowed values: PROPOSAL | COMMIT | UNRESOLVED | ABEND
- Constraint: OUT_STATE MUST match the top-level STATE field defined in 2PLT_40_OUTPUT_SCHEMA.

### ARTIFACT_CLASS
- Allowed values: LOGICAL | PHYSICAL
- LOGICAL: Inline, declarative artifacts (e.g., diff/patch text, proposals) with no physical mutation implied.
- PHYSICAL: Artifacts that include a physically mutated snapshot (e.g., ZIP) or otherwise represent an applied change.

### ARTIFACT_FORMAT
- Allowed values: INLINE | ZIP
- INLINE: The primary artifact payload is inline within the response envelope.
- ZIP: The primary artifact payload is a ZIP artifact produced/updated by the WORKER.

## Canonical Pairing (Default)

The following default pairing is normative unless explicitly overridden by a profile:

- OUT_STATE=PROPOSAL  => ARTIFACT_CLASS=LOGICAL  AND ARTIFACT_FORMAT=INLINE
- OUT_STATE=COMMIT    => ARTIFACT_CLASS=PHYSICAL AND ARTIFACT_FORMAT=ZIP
- OUT_STATE=UNRESOLVED => ARTIFACT_CLASS=PHYSICAL AND ARTIFACT_FORMAT=ZIP
- OUT_STATE=ABEND     => ARTIFACT_CLASS=LOGICAL  AND ARTIFACT_FORMAT=INLINE


Note: COMMIT responses MAY still include inline NOTES/SUMMARY for audit and guidance; ARTIFACT_FORMAT denotes the primary artifact channel.

## Rationale

These fields exist because INLINE outputs may become inputs to subsequent steps.
Explicit TRIGGER / IN_STATE / OUT_STATE and artifact typing reduces ambiguity and prevents
PROPOSAL from performing physical mutation without explicit authorization.

---

End of Document
