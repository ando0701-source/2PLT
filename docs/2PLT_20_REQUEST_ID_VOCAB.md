# 2PLT_REQUEST_ID_VOCAB (Normative)

## 1. Purpose

This document defines the machine-checkable constraints and semantics for `REQUEST_ID`, a MANAGER-supplied
idempotency key used to:
- correlate requests and responses,
- support safe retries without duplicate physical persistence, and
- strengthen auditability across terminals.

`REQUEST_ID` is an **opaque identifier**: it is not interpreted for business meaning.

## 2. Reserved Identifiers (Normative)

The following identifiers are reserved (case-insensitive) and MUST NOT be used as REQUEST_ID:

- NONE
- NULL
- NIL
- UNSET
- DEFAULT
- 0

If a reserved identifier is provided, WORKER MUST treat it as invalid input.

## 3. Syntax Constraints (Normative)

A REQUEST_ID MUST satisfy all of the following constraints:

- Non-empty after trimming.
- Length: 1–64 characters.
- Regex: `^[A-Za-z0-9][A-Za-z0-9._:-]{0,63}$`
  - (no whitespace; safe for filenames and JSON keys)

## 4. Uniqueness and Retry Semantics (Normative)

Within a session partitioned by `(OWNER_ID, LANE_ID)`:

- REQUEST_ID MUST uniquely identify one logical request attempt chain.
- If MANAGER retries the same logical request, MANAGER MUST re-use the same REQUEST_ID.
- If MANAGER initiates a new logical request, MANAGER MUST use a new REQUEST_ID.

These norms enable deterministic deduplication by downstream monitoring/audit agents.

## 5. Echo and Persistence (Normative)

- WORKER MUST echo REQUEST_ID in ARTIFACT_META for every terminal response (DOC_ID `2PLT_20_ARTIFACT_META_VOCAB`).
- When UNRESOLVED produces physical unresolved logs, both request and response unresolved records MUST include REQUEST_ID.
