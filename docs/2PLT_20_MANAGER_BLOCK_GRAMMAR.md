# 2PLT_MANAGER_BLOCK_GRAMMAR (Normative)

## 1. Purpose

This document defines the **machine-checkable grammar** and deterministic parsing rules for the
MANAGER→WORKER handover block ("MANAGER block").

It exists to make activation, trigger resolution, profile selection, and payload extraction **deterministic**.

## 2. MANAGER Block Boundary

A MANAGER block is delimited by **boundary lines**:

- `BEGIN_MANAGER`
- `END_MANAGER`

Normative constraints:

- Boundary lines are recognized only when the trimmed line equals the exact boundary token.
- A user message MAY contain boundary tokens, but MUST contain **exactly one** *parsable* MANAGER block
  for deterministic operation.

Deterministic parsing:

1. If no `BEGIN_MANAGER` token exists → **not activated** (NOOP per DOC_ID `2PLT_10_STATE_MACHINE`).
2. If `BEGIN_MANAGER` exists but no matching `END_MANAGER` after it → **activated** and MUST resolve to a terminal
   (typically ABEND) with REASON_CODE=`EXECUTION_IMPOSSIBLE`.
3. If more than one complete MANAGER block exists → **activated** and MUST resolve deterministically with
   REASON_CODE=`EXECUTION_IMPOSSIBLE` (required: provide exactly one MANAGER block).

## 3. Trigger Token Resolution

Trigger tokens are interpreted **only inside** the MANAGER block.

### 3.1 Recognition Rule (Strict)

A trigger token is recognized only when a line inside the MANAGER block satisfies:

- `trim(line) == <token>`

Where `<token>` is any `canonical_token` or `alias` in DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.

### 3.2 Deterministic Resolution Algorithm

Given a parsable MANAGER block:

1. Collect all recognized trigger tokens (per 3.1).
2. If the set is empty → REASON_CODE=`TRIGGER_INVALID`.
3. If the set contains more than one distinct token → REASON_CODE=`TRIGGER_INVALID` (ambiguous).
4. Otherwise, map the single token to `trigger_id` via DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.
   - WORKER MUST emit `TRIGGER` in canonical form in ARTIFACT_META (DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`).

## 4. MANAGER Directives


The MANAGER block may contain **directives** that affect execution deterministically.

### 4.1 OWNER_ID directive (Required)

Exactly one owner identifier MUST be declared in every parsable MANAGER block:

- `OWNER_ID: <owner_id>`

Constraints (machine-checkable):

- The directive line is recognized only when `trim(line)` starts with `OWNER_ID:` exactly (case-sensitive).
- `<owner_id>` MUST be non-empty after trimming.
- `<owner_id>` MUST match regex: `^[A-Za-z][A-Za-z0-9_]{0,31}$` (1–32 chars; starts with a letter).
- `<owner_id>` MUST NOT be a reserved identifier as defined in DOC_ID `2PLT_20_OWNER_ID_VOCAB` (case-insensitive match).
  - If reserved → REASON_CODE=`OWNER_ID_RESERVED`
- The MANAGER block MUST contain exactly one `OWNER_ID:` directive line.
  - If missing → REASON_CODE=`OWNER_ID_MISSING`
  - If more than one exists → REASON_CODE=`OWNER_ID_INVALID`
  - If `<owner_id>` violates the regex → REASON_CODE=`OWNER_ID_INVALID`

Deterministic semantics:

- `OWNER_ID` identifies the logical execution owner/route selected by MANAGER (e.g., which WORKER endpoint is intended).
- `OWNER_ID` scopes all session-dependent decisions together with `LANE_ID`, including proposal→commit linkage.
- `OWNER_ID` MUST be echoed in ARTIFACT_META (DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`) for every terminal response.

### 4.2 LANE_ID directive (Required)

Exactly one lane identifier MUST be declared in every parsable MANAGER block:

- `LANE_ID: <lane_id>`

Constraints (machine-checkable):

- The directive line is recognized only when `trim(line)` starts with `LANE_ID:` exactly (case-sensitive).
- `<lane_id>` MUST be non-empty after trimming.
- `<lane_id>` MUST match regex: `^[A-Za-z][A-Za-z0-9_]{0,31}$` (1–32 chars; starts with a letter).
- The MANAGER block MUST contain exactly one `LANE_ID:` directive line.
  - If missing → REASON_CODE=`LANE_ID_MISSING`
  - If more than one exists → REASON_CODE=`LANE_ID_INVALID`
  - If `<lane_id>` violates the regex → REASON_CODE=`LANE_ID_INVALID`

Deterministic semantics:

- `LANE_ID` scopes all session-dependent decisions together with `OWNER_ID`, including proposal→commit linkage (see DOC_ID `2PLT_40_EXECUTION_POLICY`).
- `LANE_ID` MUST be echoed in ARTIFACT_META (DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`) for every terminal response.

### 4.3 REQUEST_ID directive (Required)

Exactly one request identifier MUST be declared in every parsable MANAGER block:

- `REQUEST_ID: <request_id>`

Constraints (machine-checkable):

- The directive line is recognized only when `trim(line)` starts with `REQUEST_ID:` exactly (case-sensitive).
- `<request_id>` MUST be non-empty after trimming.
- `<request_id>` MUST match regex: `^[A-Za-z0-9][A-Za-z0-9._:-]{0,63}$` (1–64 chars; safe for filenames).
- `<request_id>` MUST NOT be a reserved identifier as defined in DOC_ID `2PLT_20_REQUEST_ID_VOCAB` (case-insensitive match).
  - If reserved → REASON_CODE=`REQUEST_ID_RESERVED`
- The MANAGER block MUST contain exactly one `REQUEST_ID:` directive line.
  - If missing → REASON_CODE=`REQUEST_ID_MISSING`
  - If more than one exists → REASON_CODE=`REQUEST_ID_INVALID`
  - If `<request_id>` violates the regex → REASON_CODE=`REQUEST_ID_INVALID`

Deterministic semantics:

- `REQUEST_ID` is an idempotency key chosen by MANAGER.
- `REQUEST_ID` does NOT change trigger resolution or proposal→commit linkage (which remains scoped by OWNER_ID and LANE_ID).
- `REQUEST_ID` MUST be echoed in ARTIFACT_META (DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`) for every terminal response.

### 4.4 PROFILE_DOC_ID directive (Optional)


At most one directive line is permitted:

- `PROFILE_DOC_ID: <doc_id>`

Constraints:

- `<doc_id>` MUST exist in DOC_ID `2PLT_05_DOC_ID_VOCAB`.
- If present and invalid, apply REASON_CODE=`SCHEMA_MISSING_REQUIRED`.

Compatibility constraints (deterministic):

- If the resolved trigger_id is `JL_PROPOSAL`, the only permitted profile is `2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL`.
- If the resolved trigger_id is `JL_COMMIT`, the only permitted profile is `2PLT_50_PROFILE_JUDGEMENT_LOG_COMMIT`.

If a PROFILE_DOC_ID is present but incompatible with the resolved trigger_id, treat as REASON_CODE=`EXECUTION_IMPOSSIBLE`
and include REQUIRED_TO_RESOLVE.


## 5. Payload Extraction

Payload is defined as the sequence of lines inside the MANAGER block after removing:

- boundary lines (`BEGIN_MANAGER`, `END_MANAGER`)
- recognized trigger-token lines (per section 3)
- directive lines (per section 4)

Payload is passed to the active profile (if any) as the **input text** for schema checks and proposal/commit obligations.

## 6. Notes

- Unknown `KEY: VALUE` lines are treated as payload (not directives), unless explicitly specified in this document.
- The grammar is intentionally minimal; future extensions MUST be added here (Layer 20) and referenced by DOC_ID.

---

End of Document
