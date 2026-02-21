# 2PLT_EXECUTION_POLICY (Normative)

## 1. Purpose

This policy defines deterministic execution obligations for activated 2PLT turns, including:
- profile selection
- proposal→commit linkage (session rule)
- when UNRESOLVED is permitted
- fail-safe escalation behavior

## 2. Profile Selection


### 2.1 Deterministic Profile Resolution (Normative)

Profile selection MUST be deterministic. Apply the following precedence, selecting exactly one active profile:

1) **Explicit profile declaration** inside the MANAGER block (if present and valid per DOC_ID `2PLT_05_DOC_ID_VOCAB`).
2) **Default-by-trigger mapping**:
   - trigger_id `JL_PROPOSAL` → profile DOC_ID `2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL`
   - trigger_id `JL_COMMIT`   → profile DOC_ID `2PLT_50_PROFILE_JUDGEMENT_LOG_COMMIT`
   - trigger_id `JL_REJECT`  → profile DOC_ID `2PLT_50_PROFILE_JUDGEMENT_LOG_REJECT`
3) If no profile is resolved by (1) or (2), operate under **no active profile** and apply only base norms:
   `2PLT_10_STATE_MACHINE`, `2PLT_30_TRIGGER_TERMINAL_MATRIX`, `2PLT_40_OUTPUT_SCHEMA`.

If an explicit profile declaration is present, it is parsed per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`.

If an explicit profile declaration is present but **invalid** (doc_id not found in DOC_ID `2PLT_05_DOC_ID_VOCAB`), treat as a schema failure:
- REASON_CODE=`SCHEMA_MISSING_REQUIRED`
- Resolve the terminal deterministically using the matrix + UNRESOLVED gate:
  - If trigger_type=COMMIT and a schema-compliant UNRESOLVED can be produced → UNRESOLVED
  - Otherwise → ABEND
In all cases, REQUIRED_TO_RESOLVE MUST be actionable (e.g., provide a valid PROFILE_DOC_ID or remove the directive).

If an explicit profile declaration is present but **incompatible** with the resolved trigger_id, treat as:
- REASON_CODE=`EXECUTION_IMPOSSIBLE`
- Resolve terminal deterministically as above (UNRESOLVED preferred only when permitted).


A profile MUST be resolved deterministically (see section 2.1).

If no explicit profile is selected, the default behavior MUST still follow:
- 2PLT_10_STATE_MACHINE
- 2PLT_30_TRIGGER_TERMINAL_MATRIX
- 2PLT_40_OUTPUT_SCHEMA

Profiles MUST NOT redefine trigger vocabularies; they may only:
- restrict trigger subsets (allowlist)
- tighten obligations and artifacts
- add additional policy constraints



## 2.2 Lane Scoping (Normative)

`LANE_ID` (declared in the MANAGER block per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`) is a required session partition key.

Norms:

- All session-dependent rules (including proposal→commit linkage and "most recent proposal" resolution) MUST be evaluated **within the same LANE_ID**.
- WORKER MUST echo `LANE_ID` in ARTIFACT_META for every terminal response (DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`).
- If the MANAGER block is present but `LANE_ID` is missing or invalid, treat as a recoverable schema failure:
  - REASON_CODE=`LANE_ID_MISSING` or `LANE_ID_INVALID`
  - Resolve terminal deterministically using the matrix + UNRESOLVED gate (UNRESOLVED preferred only when permitted).

## 2.3 REQUEST_ID (Normative)

`REQUEST_ID` (declared in the MANAGER block per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`) is a required idempotency key.

Norms:

- WORKER MUST echo `REQUEST_ID` in ARTIFACT_META for every terminal response (DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`).
- If the MANAGER block is present but `REQUEST_ID` is missing or invalid, treat as a recoverable schema failure:
  - REASON_CODE=`REQUEST_ID_MISSING`, `REQUEST_ID_INVALID`, or `REQUEST_ID_RESERVED`
  - Resolve terminal deterministically using the matrix + UNRESOLVED gate (UNRESOLVED preferred only when permitted).
- REQUEST_ID MUST be treated as opaque; it MUST NOT alter trigger resolution or proposal→commit linkage.
- For physical persistence (COMMIT / UNRESOLVED), REQUEST_ID SHOULD be included in any persisted log records to support deduplication by audit/monitor agents.

## 3. Proposal→Commit Linkage (Session Rule)

Session scope: A session is partitioned by (OWNER_ID, LANE_ID) for all linkage and monitoring decisions.

For COMMIT-type triggers, COMMIT MUST bind to the most recent PROPOSAL in the same owner (OWNER_ID) and same lane (LANE_ID) within the same session,
unless the profile defines a stricter linkage rule.

If no prior proposal exists in-session and the commit requires proposal linkage, treat as failure and apply section 5.

## 4. UNRESOLVED Permission (Deterministic Gate)

UNRESOLVED is permitted ONLY when:

- the trigger_type is COMMIT (per DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX`), AND
- the failure is **RECOVERABLE** (RECOVERY_CLASS=RECOVERABLE per DOC_ID `2PLT_20_REASON_CODE_VOCAB`), AND
- WORKER can produce a schema-compliant UNRESOLVED output (DOC_ID `2PLT_40_OUTPUT_SCHEMA`).

If these conditions hold, the system MUST select UNRESOLVED.

If UNRESOLVED is permitted by matrix but a schema-compliant UNRESOLVED output cannot be produced, the system MUST resolve to ABEND.

UNRESOLVED is the failure terminal intended to emit an actionable unresolved record (see REQUIRED_TO_RESOLVE in DOC_ID `2PLT_40_OUTPUT_SCHEMA`).
ABEND is the fail-safe failure terminal used only when UNRESOLVED cannot be emitted or when the failure is FATAL.

## 5. Escalation (Fail-Safe)

- If obligations for COMMIT cannot be satisfied:
  - select UNRESOLVED if permitted by section 4
  - otherwise select ABEND

- If obligations for PROPOSAL cannot be satisfied:
  - select ABEND

## 6. Physical Mutation Gate

Physical mutation is permitted ONLY when:
- STATE is COMMIT or UNRESOLVED, and
- ARTIFACT is produced/updated as required by the selected profile (or default policy) and DOC_ID `2PLT_40_OUTPUT_SCHEMA`.

Otherwise, physical mutation is prohibited.

### Physical Failure Escalation Rule

If terminal candidate = COMMIT and physical responsibility execution fails,
the system MUST resolve to UNRESOLVED.

If terminal candidate = UNRESOLVED and physical responsibility execution fails,
the system MUST resolve to ABEND.

Physical failure MUST NOT directly resolve to ABEND
unless UNRESOLVED resolution is structurally impossible or UNRESOLVED itself fails.

### Output Schema Violation Handling

Output schema validation MUST follow DOC_ID 2PLT_40_OUTPUT_SCHEMA.

If an output fails schema validation for its candidate terminal:

1. Candidate terminal = COMMIT:
   - The system MUST attempt to resolve to UNRESOLVED with a valid schema-compliant output.
   - If a valid UNRESOLVED output cannot be produced, the system MUST resolve to ABEND.

2. Candidate terminal = UNRESOLVED:
   - The system MUST resolve to ABEND.
   - ABEND output MUST satisfy the minimal ABEND guarantee in DOC_ID 2PLT_40_OUTPUT_SCHEMA.

3. Candidate terminal = ABEND:
   - The system MUST still produce at minimum: STATE = ABEND.
   - Any additional fields are OPTIONAL.

### Proposal Input Handling

For PROPOSAL-type triggers, if required input payload is missing or empty per the active Profile,
the system MUST terminate with terminal = ABEND.

## 7. Structural Validation (Pre-flight)

For any activated turn in STRICT_OPERATION_MODE, WORKER MUST run a structural validation pass before emitting
a non-fail terminal (PROPOSAL or COMMIT).

Structural validation MUST check (at minimum):

- Trigger validity and profile selection
- Terminal obligations per DOC_ID `2PLT_40_OUTPUT_SCHEMA`
- Profile-specific required elements (e.g., input_zip, patch_target, diff for JL_PROPOSAL)
- Ownership-boundary validity per DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` (role-owned definitions only)
- Forbidden ambiguous-term scan for normative requirements per DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE`

If any required element is missing or non-mechanically-applicable, WORKER MUST terminate immediately with a RECOVERABLE failure:

- For COMMIT-type triggers: MUST select UNRESOLVED (REASON_CODE=INPUT_MISSING) when a schema-compliant UNRESOLVED output can be produced.
- Otherwise: MUST select ABEND (REASON_CODE=INPUT_MISSING).

In both cases, REQUIRED_TO_RESOLVE MUST be included (RECOVERY_CLASS=RECOVERABLE).

## 8. Failure-State Precedence

Failure states dominate success states.

If a success terminal (PROPOSAL/COMMIT) would require bypassing any MUST requirement, WORKER MUST choose ABEND
(or UNRESOLVED only when explicitly permitted) instead of producing a partial success output that violates any MUST requirement.

## 9. "Cannot be Determined" Interpretation

When a requirement says an element must be "determined", it MUST be explicitly present or resolvable under an
explicit resolution rule. Semantic inference and template completion are prohibited.

Undefined placeholders (e.g., "〇〇") count as missing input when used where a concrete identifier is required.

## 10. Audit Checks (Mechanizable)

This section defines a deterministic verification order for an activated turn. It exists to enable an independent audit agent to validate compliance without relying on internal reasoning traces.

On failure (STATE in {UNRESOLVED, ABEND}), REQUIRED_TO_RESOLVE MUST follow the Self-Repair Feedback Governance in DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` (section 10).

**Evaluation Order Constraint:** The order below is a normative verification order. It does not constrain internal reasoning order.

### 10.1 Pre-check: Activated Turn Parsing
1) Parse MANAGER block boundaries and content per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`.
2) If the MANAGER block is missing or invalid, treat as a schema failure and resolve deterministically (UNRESOLVED only when permitted; otherwise ABEND).

### 10.2 Identity + Trigger Resolution
3) Validate exactly one trigger token (DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`), exactly one OWNER_ID (DOC_ID `2PLT_20_OWNER_ID_VOCAB`), exactly one LANE_ID, and exactly one REQUEST_ID (DOC_ID `2PLT_20_REQUEST_ID_VOCAB`).
4) Resolve `trigger_id` and `trigger_type` from the trigger token (see DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`).
   Note: MANAGER construction rules require canonical token emission (DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE`, section 11).

### 10.3 Profile Resolution
5) Resolve the active profile deterministically per section 2.1 (explicit PROFILE_DOC_ID if valid; else default-by-trigger; else none).

### 10.4 Terminal Candidate + Permission
6) Determine the candidate terminal obligations for the resolved trigger_type (success terminal = PROPOSAL or COMMIT) and verify the terminal is permitted by DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX`.
7) If any MUST obligation for the success terminal cannot be satisfied, apply escalation (section 5) and the UNRESOLVED gate (section 4).

### 10.5 Output Contract Validation
8) Validate the output envelope and terminal-specific obligations per DOC_ID `2PLT_40_OUTPUT_SCHEMA`.
9) Enforce Strict Output Self-Description (section 11) and validate ARTIFACT_META values against:
   - MANAGER block values (OWNER_ID, LANE_ID, REQUEST_ID)
   - terminal pairing rules (ARTIFACT vs ARTIFACT_FORMAT)
   - canonical trigger token rules
   - IN_STATE mapping rules (DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`: deterministic mapping by trigger_type and OUT_STATE)

### 10.6 Physical Mutation + Write-Scope Validation
10) If physical mutation occurred, verify it is permitted (section 6) and limited to the write-scope:
    `owners/<OWNER_ID>/lanes/<LANE_ID>/` (profile-specific rules may further restrict).
11) If physical mutation is required (COMMIT / UNRESOLVED) but not present or not valid, apply Physical Failure Escalation Rule (section 6).

### 10.7 Linkage Validation (COMMIT)
12) For COMMIT-type triggers, validate proposal→commit linkage within `FLOW_KEY=(OWNER_ID, LANE_ID)` per section 3 (or stricter profile rule).
13) If linkage is required but missing, resolve deterministically using section 5 + section 4 (UNRESOLVED preferred only when permitted).

The above checks are sufficient to validate the protocol invariants defined by DOC_ID `2PLT_00_MODEL` (Protocol Invariants).
---

## 11. Strict Output Self-Description (Normative)

In STRICT_OPERATION_MODE, the WORKER MUST emit the artifact metadata fields defined in DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`.
This requirement exists to prevent role/state confusion when INLINE output becomes future input.



### Audit Check: JL_REJECT terminal and artifacts
- If trigger_id=JL_REJECT then OUT_STATE MUST be UNRESOLVED or ABEND and MUST follow profile DOC_ID `2PLT_50_PROFILE_JUDGEMENT_LOG_REJECT`.
