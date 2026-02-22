# 2PLT Model (Authoritative)

## BOOT LOADER (Normative)

The document set to be consulted is determined at boot time by selecting a BOOT LOADER JSON using the trigger token.
The selected BOOT LOADER JSON provides a physical-path-only load order, and activated processing MUST remain within the BOOT-loaded document set.


## 0. Normative Scope

This document defines the authoritative 2PLT framework and its interpretation rules.

The following documents are **normative** and have binding force:

### Meta Governance Layer (framework-invariant)
- 2PLT_00_DOCUMENT_GOVERNANCE
- 2PLT_00_ENTRYPOINT
- 2PLT_05_DOC_ID_VOCAB


### Core Semantics Layer (framework-invariant)
- 2PLT_10_STATE_MACHINE
- 2PLT_10_RESPONSIBILITY

### Lexical Layer (closed vocabularies)
- 2PLT_20_TRIGGER_ID_VOCAB
- 2PLT_20_ARTIFACT_META_VOCAB
- 2PLT_20_REASON_CODE_VOCAB
- 2PLT_20_OWNER_ID_VOCAB
- 2PLT_20_REQUEST_ID_VOCAB
- 2PLT_20_MANAGER_BLOCK_GRAMMAR
### Mapping Layer (fixed semantics mapping)
- 2PLT_30_TRIGGER_TERMINAL_MATRIX

### Policy Layer (execution + output contract)
- 2PLT_40_EXECUTION_POLICY
- 2PLT_40_OUTPUT_SCHEMA
- 2PLT_40_AUDIT_CHECKS

### Profile Layer (use-case specializations; optional)
- 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL
- 2PLT_50_PROFILE_JUDGEMENT_LOG_COMMIT
- 2PLT_50_PROFILE_JUDGEMENT_LOG_REJECT




### Purpose

2PLT is a reusable control framework to reliably steer a cooperating generator (or subordinate agent) toward exactly one of four external terminal outcomes per activated turn:

- PROPOSAL
- COMMIT
- UNRESOLVED
- ABEND

It guarantees:

Note: MANAGER MUST emit canonical trigger tokens only (see DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE`, section 11).



1. Deterministic activation boundary (see DOC_ID `2PLT_10_STATE_MACHINE`)
2. Deterministic terminal set (exactly one terminal per activated turn; see DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX`)
3. Hard bans outside defined transitions (see DOC_ID `2PLT_10_STATE_MACHINE`)
4. Separation of logical vs physical responsibility (see DOC_ID `2PLT_10_RESPONSIBILITY`, DOC_ID `2PLT_40_EXECUTION_POLICY`)
5. Portability via Profiles (use-case specialization without mutating core semantics; see DOC_ID `2PLT_50_PROFILE_*`)

### Design Principles (Framework Quality Targets)

- **Closure:** The framework defines a closed set of external terminals and failure escalation paths.
- **Determinism:** For an activated turn, the system MUST select exactly one terminal outcome.
- **Separation of concerns:** Mapping (trigger→terminal), vocabularies, policy, and profiles are distinct layers with explicit ownership.
- **Reusability:** Profiles specialize policy and artifacts without rewriting the state machine.
- **Auditability:** Commit actions (physical responsibility) MUST be explicit and verifiable.

### Definitions

- **Activated turn:** A turn that satisfies activation boundary conditions (see DOC_ID `2PLT_10_STATE_MACHINE`).
- **MANAGER block:** The bounded handover block `BEGIN_MANAGER ... END_MANAGER` parsed deterministically by DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`.
- **Owner (OWNER_ID):** A logical execution owner/route selected by MANAGER, declared by `OWNER_ID: <owner_id>` inside the MANAGER block (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR` and DOC_ID `2PLT_20_OWNER_ID_VOCAB`). Together with LANE_ID, it scopes all session-dependent decisions.
- **Lane (LANE_ID):** A logical execution lane (session partition key) declared by `LANE_ID: <lane_id>` inside the MANAGER block (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`). All session-dependent decisions (e.g., proposal→commit linkage) are scoped to a single lane.
- **Trigger token:** A textual marker that maps to a `trigger_id` (see DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`).
- **Trigger type:** `PROPOSAL` or `COMMIT` (lexically defined in DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`).
- **IN_STATE:** The source state of the final transition edge that produced OUT_STATE in this activated turn (see DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`). `NUL` means start-of-turn (no predecessor state).
- **BOOTSTRAP / ACK:** A non-activated prelude handshake. It is NOT a terminal outcome in `{PROPOSAL, COMMIT, UNRESOLVED, ABEND}`. Handling rules are defined by DOC_ID `2PLT_10_STATE_MACHINE` (section 3.1).
- **Terminal outcome:** One of `{PROPOSAL, COMMIT, UNRESOLVED, ABEND}`.
- **Policy:** Deterministic rules that decide content obligations and when `UNRESOLVED` is permissible (see DOC_ID `2PLT_40_EXECUTION_POLICY` and optionally DOC_ID `2PLT_50_PROFILE_*`).

### External Interface Contract (Normative)

The MANAGER→WORKER handover for an activated turn is expressed by including (a) exactly one valid trigger token, (b) exactly one required `OWNER_ID: <owner_id>` directive (valid per DOC_ID `2PLT_20_OWNER_ID_VOCAB`), and (c) exactly one required `LANE_ID: <lane_id>` directive inside the MANAGER block boundaries, using the parsing rules defined by DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR` (activation semantics per DOC_ID `2PLT_10_STATE_MACHINE`).

- Valid trigger tokens (canonical or alias) are defined ONLY by DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.
- The trigger token determines `trigger_id` and `trigger_type` (`PROPOSAL` or `COMMIT`).
- The allowed terminal outcome set is determined ONLY by `trigger_type` and DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX`.

Trigger tokens are defined ONLY by DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.

- **Canonical tokens are normative.** When MANAGER emits a MANAGER block, it MUST use **canonical tokens only**
  (DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE`, section 11).
- **Aliases** MAY exist for compatibility / human-authored inputs. WORKER-side parsing MAY normalize known aliases
  to the canonical token, but this MUST NOT relax the MANAGER generation requirement.

(Informative) Canonical examples:
- `@@@@2PLT_JL_PROPOSAL@@@@` → trigger_id `JL_PROPOSAL` → trigger_type `PROPOSAL`
- `@@@@2PLT_JL_COMMIT@@@@` → trigger_id `JL_COMMIT` → trigger_type `COMMIT`
- `@@@@2PLT_JL_REJECT@@@@` → trigger_id `JL_REJECT` → trigger_type `COMMIT`

`UNRESOLVED` is permitted only when it is allowed by the matrix AND the deterministic UNRESOLVED gate in DOC_ID `2PLT_40_EXECUTION_POLICY` is satisfied; otherwise the system MUST escalate to `ABEND`.


Optional Profile Declaration (deterministic):

- The MANAGER block MAY include exactly one line of the form:
  - `PROFILE_DOC_ID: <doc_id>`
- If present, `<doc_id>` MUST be a valid document id in DOC_ID `2PLT_05_DOC_ID_VOCAB`.
- Profile resolution precedence is defined by DOC_ID `2PLT_40_EXECUTION_POLICY` (section 2.1).


### Protocol Invariants (Normative)

The following invariants define the framework's non-negotiable external behavior. If any invariant cannot be satisfied, WORKER MUST select a failure terminal (UNRESOLVED only when permitted; otherwise ABEND) as defined by DOC_ID `2PLT_40_EXECUTION_POLICY`.

- **INV-01 Activation determinism:** An activated turn is recognized ONLY by a schema-valid MANAGER block per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR` and DOC_ID `2PLT_10_STATE_MACHINE`.
- **INV-02 Single trigger + identity tuple:** In an activated turn, exactly one trigger token, exactly one `OWNER_ID`, exactly one `LANE_ID`, and exactly one `REQUEST_ID` MUST be present and valid (DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`, `2PLT_20_OWNER_ID_VOCAB`, `2PLT_20_REQUEST_ID_VOCAB`, `2PLT_20_MANAGER_BLOCK_GRAMMAR`).
- **INV-03 Closed terminals + single terminal:** For an activated turn, WORKER MUST emit exactly one terminal outcome from `{PROPOSAL, COMMIT, UNRESOLVED, ABEND}` and it MUST be permitted by DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX` (selection details in DOC_ID `2PLT_40_EXECUTION_POLICY`).
- **INV-04 Deterministic profile selection:** Profile resolution MUST be deterministic and follow DOC_ID `2PLT_40_EXECUTION_POLICY` (profiles MUST NOT mutate core semantics).
- **INV-05 Proposal→commit linkage scope:** Any proposal→commit linkage and "most recent proposal" resolution is scoped to `FLOW_KEY=(OWNER_ID, LANE_ID)` (DOC_ID `2PLT_40_EXECUTION_POLICY`).
- **INV-06 Physical mutation boundary:** Physical mutation is permitted ONLY for COMMIT and UNRESOLVED, and ONLY within the write-scope owned by `owners/<OWNER_ID>/lanes/<LANE_ID>/` (DOC_ID `2PLT_10_RESPONSIBILITY`, `2PLT_40_EXECUTION_POLICY`, `2PLT_50_PROFILE_*`).
- **INV-07 Output contract + self-description:** The output envelope MUST satisfy DOC_ID `2PLT_40_OUTPUT_SCHEMA`, and WORKER MUST emit artifact metadata with internally consistent values per DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`.
- **INV-08 "Cannot be determined":** Any required element must be explicitly present or resolvable under an explicit rule; semantic inference is prohibited (DOC_ID `2PLT_40_EXECUTION_POLICY`).

The mechanizable audit procedure for checking these invariants is defined by DOC_ID `2PLT_40_AUDIT_CHECKS`, using semantics from DOC_ID `2PLT_40_EXECUTION_POLICY` and DOC_ID `2PLT_40_OUTPUT_SCHEMA`.

### Doc Loading Policy (Normative; Fast Path)

To reduce variance and enable unit-testable behavior, activated processing MUST follow a **minimal doc loading policy**.

#### document-set (MANAGER payload key)

In an activated MANAGER block, MANAGER MUST provide a payload line:

`document-set` is treated as payload (see DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`, Notes) and has normative semantics defined here.

WORKER MUST:

2. Consult ONLY documents in that set for normative interpretation.
3. MUST NOT scan or read unrelated documents (including `etc/*`) during activated processing.

If `document-set` is missing or invalid, WORKER MUST terminate with ABEND (REASON_CODE=`SCHEMA_MISSING_REQUIRED`) and include REQUIRED_TO_RESOLVE.

WORKER MUST resolve `<doc_set_id>` using that vocabulary.

Note: `2PLT_40_AUDIT_CHECKS` is normative for evaluation/self-check, but is NOT required for WORKER execution under the fast path.

### Artifact Metadata (Normative)

The WORKER output MUST include the following metadata fields, as defined in DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`:

- TRIGGER
- OWNER_ID
- LANE_ID
- REQUEST_ID
- IN_STATE
- OUT_STATE
- ARTIFACT_CLASS
- ARTIFACT_FORMAT


These fields strengthen STRICT operation by binding State/Physical/Profile requirements and by making INLINE outputs self-describing.


### Precedence (conflict resolution)

If any conflict arises, precedence MUST follow the numeric `layer` order defined in DOC_ID `2PLT_05_DOC_ID_VOCAB`.

Normative interpretation order (highest precedence first):

1. **Layer 00** (MODEL / META) — DOC_ID `2PLT_00_MODEL`, DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE`
2. **Layer 05** (META) — DOC_ID `2PLT_05_DOC_ID_VOCAB`
3. **Layer 10** (CORE_SEMANTICS) — DOC_ID `2PLT_10_STATE_MACHINE`, DOC_ID `2PLT_10_RESPONSIBILITY`
4. **Layer 20** (LEXICAL) — DOC_ID `2PLT_20_*_VOCAB`
5. **Layer 30** (MAPPING) — DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX`
6. **Layer 40** (POLICY) — DOC_ID `2PLT_40_EXECUTION_POLICY`, DOC_ID `2PLT_40_OUTPUT_SCHEMA`
7. **Layer 50** (PROFILE) — DOC_ID `2PLT_50_PROFILE_*`, if present

#### Formal Layer Comparison (Mechanizable)

Let:
- `layer(DOC_ID)` be the integer `layer` value from DOC_ID `2PLT_05_DOC_ID_VOCAB`.
- `key(DOC_ID) = (layer(DOC_ID), DOC_ID)`.

Define strict precedence between documents:
- `DOC_A ≺ DOC_B`  **iff**  `key(DOC_A) <lex key(DOC_B)` (lexicographic order).

Interpretation:
- Smaller `layer` value means **higher** precedence.
- Within the same layer, DOC_ID lexical order defines a deterministic **evaluation order only**.
- Normative contradictions between documents in the **same** layer are **prohibited** and MUST be treated as a structural violation.
  The system MUST terminate with a fail terminal (ABEND preferred) under the policy failure rules.
