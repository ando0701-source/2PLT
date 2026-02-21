# 2PLT Document Governance (Normative)

This document defines document-level handling rules for the 2PLT framework.

## 1. DOC_ID Referencing Rule

All cross-document references MUST use DOC_ID form.

Direct filename references (e.g., *.md) are strictly prohibited.

Rationale:
- Ensures stability under file renaming or restructuring.
- Guarantees abstraction layer between logical identity and physical filename.

## 2. Binding Authority

Normative documents are those listed in DOC_ID 2PLT_00_MODEL under Normative Scope.

Non-listed documents have no binding force.

## 3. Modification Constraints

- Core layer documents MUST NOT be modified by Profiles.
- Policy may specialize but not mutate Core semantics.
- Profiles may restrict but never expand terminal sets.

## 4. Conflict Resolution

Precedence order is defined in DOC_ID `2PLT_00_MODEL` (see Formal Layer Comparison).


## 5. Policy Boundary Constraints

Policy documents MUST NOT:
- introduce new terminal states (terminals are closed by DOC_ID 2PLT_00_MODEL and DOC_ID 2PLT_10_STATE_MACHINE)
- redefine trigger_type semantics (trigger_type is defined only by DOC_ID 2PLT_20_TRIGGER_ID_VOCAB)

Policy documents MAY:
- tighten obligations (e.g., stronger output constraints, stricter artifact requirements)
- tighten escalation (e.g., prefer ABEND over UNRESOLVED when allowed by matrix and core semantics)
- restrict trigger subsets via Profiles

## 6. Profile Monotonicity Constraint

Profiles MUST be monotone-tightening relative to Policy and Core.

Profiles MUST NOT:
- remove obligations defined in Policy
- relax mandatory requirements to optional
- expand allowed terminal sets beyond those defined by Mapping and Core

Profiles MAY:
- introduce stricter constraints
- restrict trigger subsets
- require additional artifacts or validation steps

## 7. Evaluation Order Constraint

Normative interpretation MUST follow this fixed order:

1. Vocabulary (DOC_ID `2PLT_05_DOC_ID_VOCAB`, DOC_ID `2PLT_20_*_VOCAB`)
2. State Machine (DOC_ID `2PLT_10_STATE_MACHINE`)
3. Execution Policy (DOC_ID `2PLT_40_EXECUTION_POLICY`)
4. Output Schema (DOC_ID `2PLT_40_OUTPUT_SCHEMA`)
5. Profile-specific constraints (DOC_ID `2PLT_50_PROFILE_*`), if present

Lower layers MUST NOT be evaluated prior to higher layers.

This constraint governs normative rule validation order, not the internal reasoning process of the model.
Internal reasoning may occur in any order, but final rule validation MUST follow the prescribed evaluation order.

## 8. Semantic Isolation Rule

Each normative document MUST define semantics only within the authority domain owned by its `role`
(see DOC_ID `2PLT_05_DOC_ID_VOCAB`).

### 8.1 Role Ownership Table (Mechanizable)

| role | owns (authoritative definitions) | MUST NOT define |
|---|---|---|
| CORE_SEMANTICS | terminals, state machine, responsibility mapping, cross-layer constraints, interpretation rules | vocab tokens, trigger→terminal mapping tables, output schema fields |
| META | doc_id binding, layer metadata, document governance rules | terminals, vocab tokens, mapping rules, policy/profile constraints |
| LEXICAL | closed vocabularies (trigger_id, reason_code, artifact_meta, etc.) | terminals, mapping rules, execution policy obligations |
| MAPPING | trigger→allowed terminal sets (matrix) | new terminals, new vocab tokens, execution behavior |
| POLICY | execution obligations, structural validation rules, output schema (required keys/fields) | new terminals, new vocab tokens, mapping changes |
| PROFILE | use-case specializations that only tighten constraints | expanding terminals, relaxing MUST requirements, redefining higher-layer semantics |

### 8.2 Mechanizable Ownership Check

A structural validation pass MUST treat the following as ownership violations:

- A POLICY or PROFILE document introducing a new terminal token.
- A POLICY or PROFILE document redefining trigger_id tokens.
- Any document introducing a new `reason_code` outside DOC_ID `2PLT_20_REASON_CODE_VOCAB`.
- Any document introducing a new `artifact_meta` field outside DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`.
- Any document redefining the trigger→terminal mapping outside DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX`.

Violations MUST be treated as structural failures and MUST NOT be resolved via partial compliance or approximation.

## 9. Structural Verifiability Principle

Normative rules MUST be expressed in a mechanically verifiable manner.

### 9.1 Forbidden Ambiguous Terms (Enumerated)

The following terms MUST NOT appear inside normative requirements (sentences containing MUST / MUST NOT / REQUIRED / PROHIBITED),
unless the same section provides explicit enumerated criteria that removes ambiguity:

- sufficient
- appropriate
- reasonable
- if possible
- as needed
- etc.

### 9.2 Compliance Guidance (Mechanizable)

When a term above is unavoidable, the document MUST:
1. replace it with an explicit boolean condition, or
2. replace it with an explicit finite list of criteria (bullet list or table).

## 10. Self-Repair Feedback Governance

This section defines how an agent MUST express actionable repair guidance when a mechanizable audit check fails.
It exists to enable deterministic self-repair loops (MANAGER ↔ WORKER ↔ AUDIT) without relying on implicit context.

### 10.1 Applicability

When an activated turn resolves to STATE in {UNRESOLVED, ABEND} due to a compliance failure (schema failure, vocab failure, mapping/policy/profile violation),
the response MUST include REQUIRED_TO_RESOLVE in the output envelope (DOC_ID `2PLT_40_OUTPUT_SCHEMA`).

### 10.2 REQUIRED_TO_RESOLVE Minimal Record (Mechanizable)

REQUIRED_TO_RESOLVE MUST be an **array of objects** (ordered by priority; first item is the primary blocking issue).


Each record MUST include:

- CHECK_ID: a stable identifier of the failing audit check (see DOC_ID `2PLT_40_EXECUTION_POLICY` / "Audit Checks").
- FAIL_REASON_CODE: a member of DOC_ID `2PLT_20_REASON_CODE_VOCAB`.
- FIX_KIND: one of {INPUT_REPAIR, SPEC_REPAIR}.
- FIX_DOC_ID: the authoritative DOC_ID to modify to resolve the failure.
- FIX_SECTION: a short section identifier or heading name within FIX_DOC_ID (string; MAY be empty only when FIX_DOC_ID is unambiguous).
- FIX_HINT: a finite, explicit instruction set (no ambiguous terms; MUST be actionable).

Constraints:
- FIX_HINT MUST enumerate missing/invalid fields or provide a finite list of valid alternatives.
- The record MUST NOT reference physical filenames; DOC_ID referencing rules apply.
- The record MUST NOT propose changes that violate Role Ownership (see section 8).

### 10.3 Fix Target Resolution Rules (Normative)

The agent MUST choose FIX_DOC_ID according to the authority domain that owns the violated rule:

| Failure class | Typical FAIL_REASON_CODE | FIX_KIND | FIX_DOC_ID (authoritative) |
|---|---|---|---|
| MANAGER block parse / missing required keys | OWNER_ID_MISSING, LANE_ID_MISSING, REQUEST_ID_MISSING, TRIGGER_INVALID | INPUT_REPAIR | `2PLT_20_MANAGER_BLOCK_GRAMMAR` |
| Trigger token not in vocabulary | TRIGGER_INVALID | INPUT_REPAIR | `2PLT_20_TRIGGER_ID_VOCAB` (if token definition is missing) OR `2PLT_20_MANAGER_BLOCK_GRAMMAR` (if input is malformed) |
| Reason code not in vocabulary | REASON_CODE_INVALID, ENUM_OUT_OF_RANGE | SPEC_REPAIR | `2PLT_20_REASON_CODE_VOCAB` |
| Output envelope / artifact meta field missing | OUTPUT_SCHEMA_VIOLATION, ARTIFACT_META_MISSING | SPEC_REPAIR | `2PLT_40_OUTPUT_SCHEMA` OR `2PLT_20_ARTIFACT_META_VOCAB` |
| Terminal not allowed by matrix | TERMINAL_NOT_ALLOWED | SPEC_REPAIR | `2PLT_30_TRIGGER_TERMINAL_MATRIX` |
| Write-scope violation | WRITE_SCOPE_VIOLATION | SPEC_REPAIR | relevant `2PLT_50_PROFILE_*` (tighten/clarify) and/or `2PLT_40_EXECUTION_POLICY` |
| Profile selection / incompatibility | EXECUTION_IMPOSSIBLE | INPUT_REPAIR (preferred) | `2PLT_20_MANAGER_BLOCK_GRAMMAR` (PROFILE_DOC_ID) or `2PLT_40_EXECUTION_POLICY` (selection rule) |

Notes:
- If a failure can be resolved purely by correcting MANAGER input, FIX_KIND MUST be INPUT_REPAIR (preferred) to minimize spec churn.
- SPEC_REPAIR is permitted only when the failure indicates a genuine spec inconsistency or missing vocabulary entry.

### 10.4 Single-Blocking-Issue Rule (Convergence)

For deterministic convergence, an agent SHOULD report only the earliest failing check in the normative evaluation order (see section 7),
unless multiple failures share the same FIX_DOC_ID and can be fixed together without ambiguity.


## 11. Manager Message Construction Governance (Normative)

This section defines **MANAGER-side construction rules** for producing valid and unambiguous MANAGER blocks.
These rules are intended to prevent token/field ambiguity and to improve machine auditability.

### 11.1 Canonical Trigger Token Rule

- The single source of truth for triggers is DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.
- MANAGER MUST emit **canonical trigger tokens only**.
- MANAGER MUST NOT emit any trigger alias token (even if the WORKER parser accepts it).

Derived canonical token table (informative; do not redefine tokens here):

| trigger_id | canonical_token |
|---|---|
| JL_PROPOSAL | @@@@2PLT_JL_PROPOSAL@@@@ |
| JL_COMMIT | @@@@2PLT_JL_COMMIT@@@@ |
| JL_REJECT | @@@@2PLT_JL_REJECT@@@@ |

### 11.2 Minimal MANAGER Block Template

MANAGER MUST construct activated requests using the following minimal template (field order is RECOMMENDED, not required):

```
BEGIN_MANAGER
OWNER_ID: <owner_id>
LANE_ID: <lane_id>
REQUEST_ID: <request_id>
<canonical_trigger_token>
PROFILE_DOC_ID: <profile_doc_id>

<payload (profile-specific)>
END_MANAGER
```

Normative constraints:

- `BEGIN_MANAGER` and `END_MANAGER` MUST appear exactly once and MUST bound the block.
- `OWNER_ID`, `LANE_ID`, and `REQUEST_ID` MUST each appear exactly once (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`).
- `PROFILE_DOC_ID` MUST be explicitly provided (do not rely on implicit mapping).
- The trigger token MUST be a canonical token (see §11.1) and MUST appear exactly once.

### 11.3 MANAGER Preflight Checks (Mechanizable)

Before sending a MANAGER block, MANAGER SHOULD perform the following deterministic checks:

1) **Boundary check**: `BEGIN_MANAGER ... END_MANAGER` are present exactly once and properly nested.
2) **Required fields**:
   - Exactly one `OWNER_ID`, `LANE_ID`, `REQUEST_ID`, and `PROFILE_DOC_ID`.
3) **Vocabulary checks**:
   - `OWNER_ID` satisfies DOC_ID `2PLT_20_OWNER_ID_VOCAB`.
   - `REQUEST_ID` satisfies DOC_ID `2PLT_20_REQUEST_ID_VOCAB`.
   - Trigger token is a **canonical token** listed in DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.
   - `PROFILE_DOC_ID` exists in DOC_ID `2PLT_05_DOC_ID_VOCAB`.
4) **Trigger/Profile compatibility**:
   - The profile selected is consistent with the trigger intent (e.g., JL_PROPOSAL → `2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL`,
     JL_COMMIT → `2PLT_50_PROFILE_JUDGEMENT_LOG_COMMIT`), unless an explicit alternative is documented.

These checks are construction-time checks for MANAGER and do not change WORKER-side parsing rules.

### 11.4 Bootstrap Handshake Prelude Construction (Normative)

This subsection defines MANAGER-side construction rules for the **BOOTSTRAP handshake prelude** used before the first activated MANAGER block.

The BOOTSTRAP handshake prelude exists to make **role and readiness** mechanically checkable **without activation**.

A BOOTSTRAP handshake prelude:

- MUST be sent in a **separate message** (not combined with any `BEGIN_MANAGER ... END_MANAGER` block).
- MUST NOT contain any trigger token (canonical or alias).
- MUST contain **only** the lines defined by the template below (no additional free text; no separators).

Minimal BOOTSTRAP handshake prelude template (field order is REQUIRED):

```
BOOTSTRAP_HANDSHAKE
BOOTSTRAP_TARGET: WORKER
BOOTSTRAP_CONTEXT: CLEAN
BOOTSTRAP_ZIP: <input_zip_filename>
BOOTSTRAP_OUTPUT: ACK_ONLY
```

Normative constraints (mechanizable):

- The first line MUST equal the exact token `BOOTSTRAP_HANDSHAKE`.
- `BOOTSTRAP_TARGET:` MUST appear exactly once and its value MUST equal `WORKER`.
- `BOOTSTRAP_CONTEXT:` MUST appear exactly once and its value MUST equal `CLEAN`.
- `BOOTSTRAP_ZIP:` MUST appear exactly once and MUST be a non-empty filename string.
- `BOOTSTRAP_OUTPUT:` MUST appear exactly once and its value MUST equal `ACK_ONLY`.
- (Construction rule) MANAGER SHOULD set `BOOTSTRAP_ZIP:` to the attached artifact filename.
- No other non-empty lines are permitted in the BOOTSTRAP handshake prelude.

After a valid BOOTSTRAP handshake prelude, MANAGER SHOULD send the first activated request using the template in section 11.2.


NOTE:
- The BOOTSTRAP handshake prelude does not activate 2PLT; activation requires a `BEGIN_MANAGER` boundary token (DOC_ID `2PLT_10_STATE_MACHINE`).

### 11.5 DOC_SET Construction Governance (Normative)

This subsection defines MANAGER-side governance for constructing `DOC_SET` in activated MANAGER blocks.

#### 11.5.1 Source of Truth

- Allowed `DOC_SET` IDs and their bound DOC_ID lists are defined only in DOC_ID `2PLT_20_DOC_SET_VOCAB`.
- Profiles SHOULD declare `DOC_SET_ID` and MAY declare `REQUIRES_DOC_ID` as a mechanizable dependency hint.

#### 11.5.2 Construction Rule

For an activated MANAGER block, MANAGER MUST:

1. Select `PROFILE_DOC_ID`.
2. Determine `<doc_set_id>` using the procedure in DOC_ID `2PLT_20_DOC_SET_VOCAB` (Manager DOC_SET Construction).
3. Include the payload line: `DOC_SET: <doc_set_id>`.

MANAGER MUST NOT omit DOC_SET in Phase1 fast path.

#### 11.5.3 Preflight Check (Mechanizable)

Before sending, MANAGER SHOULD deterministically verify:

- The selected profile document declares `DOC_SET_ID`, and the value equals the `DOC_SET` used in the payload.
- The `DOC_SET` value exists in DOC_ID `2PLT_20_DOC_SET_VOCAB`.

If the check fails, MANAGER SHOULD self-repair by updating the minimal `FIX_DOC_ID` set:
- `2PLT_20_DOC_SET_VOCAB` and/or the active profile DOC_ID.



## Doc Access Governance (Normative)

- WORKER MUST follow the entry procedure in DOC_ID `2PLT_00_ENTRYPOINT`.
- During activated processing, ZIP enumeration (listing file trees) is prohibited.
