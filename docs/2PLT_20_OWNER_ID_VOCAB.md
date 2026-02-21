# 2PLT_20_OWNER_ID_VOCAB

## Purpose

This document defines the naming rules for `OWNER_ID`, including reserved identifiers and a recommended set.
`OWNER_ID` is a **logical routing/namespace label** selected by MANAGER (e.g., which WORKER endpoint/agent instance is intended).
It is used to prevent cross-owner mixing when logs or artifacts are aggregated or when shared storage is introduced.

## Canonical Grammar (Normative)

- `OWNER_ID` MUST match regex: `^[A-Za-z][A-Za-z0-9_]{0,31}$`
  - Length: 1–32 chars
  - Starts with a letter
  - Allowed chars: letters, digits, underscore (`_`)
- Reserved-word matching is **case-insensitive**.

## Reserved OWNER_ID Identifiers (Normative; Case-Insensitive)

The following identifiers are reserved and MUST NOT be used as `OWNER_ID`:

- SYSTEM
- DEFAULT
- ROOT
- GLOBAL
- UNKNOWN
- AUTO
- NONE
- NULL
- MANAGER
- WORKER

If `OWNER_ID` matches any reserved identifier (case-insensitive), the MANAGER block is invalid and the WORKER MUST use REASON_CODE=`OWNER_ID_RESERVED`.

## Recommended OWNER_ID Set (Informational)

To reduce drift, prefer using one of the following stable identifiers:

- worker_primary
- worker_secondary
- worker_audit
- worker_monitor

Projects may extend this list by updating this document (DOC_ID `2PLT_20_OWNER_ID_VOCAB`).

## Notes (Informational)

- `OWNER_ID` is not an authentication claim; it is a protocol label used for deterministic scoping and aggregation.
- `OWNER_ID` is paired with `LANE_ID` to scope session-dependent decisions (e.g., proposal→commit linkage).
- When shared storage exists, the preferred namespace layout is:

  - `owners/<OWNER_ID>/lanes/<LANE_ID>/...`

---

End of Document
