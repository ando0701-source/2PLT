# 2PLT_ENTRYPOINT (Normative)

## Purpose

This document defines the **deterministic entry procedure** for an activated WORKER to interpret a MANAGER block
without ZIP-wide scanning.

This exists to reduce "document hunting" time and to make DOC_SET mechanizable.

## Deterministic Entry Procedure (Normative)

When activated (a parsable `BEGIN_MANAGER`…`END_MANAGER` block exists), the WORKER MUST:

1. Read this document: `2PLT_00_ENTRYPOINT` (canonical file: `2PLT_00_ENTRYPOINT.md`).
2. Read the profile document indicated by `PROFILE_DOC_ID:` (canonical file: `<DOC_ID>.md`).
3. If a payload line `DOC_SET: <doc_set_id>` exists, read `2PLT_20_DOC_SET_VOCAB` and resolve `<doc_set_id>` to an allowed DOC_ID set.
4. Consult ONLY the resolved DOC_ID set for normative interpretation (plus the active profile doc itself).

## File Resolution Rule (Normative)

For any `DOC_ID` listed above, the canonical file path is:

- `<DOC_ID>.md` at the ZIP root.

Therefore, DOC_ID lookup does NOT require ZIP enumeration.

## ZIP Enumeration Ban (Normative)

During activated processing the WORKER MUST NOT enumerate the ZIP file tree (e.g., via `namelist()`).

Permitted ZIP access is limited to:
- direct read of required canonical doc files by exact filename (`<DOC_ID>.md`),
- direct read/write of patch targets under allowed write scope when producing artifacts.

## Artifact Channel Invariants (Normative)

Unless a profile explicitly overrides:

- If `STATE=PROPOSAL`  → `ARTIFACT=INLINE`
- If `STATE=ABEND`     → `ARTIFACT=INLINE`
- If `STATE=COMMIT`    → `ARTIFACT` references a produced ZIP artifact id (filename/path)
- If `STATE=UNRESOLVED`→ `ARTIFACT` references a produced ZIP artifact id (filename/path)

Additionally:
- A PROPOSAL MUST NOT apply any patch; it proposes a mechanically-applicable `diff/patch` inline.
- A PROPOSAL MUST NOT produce any ZIP artifact.

## Output Text Format Ban (Normative)

The WORKER response envelope MUST be plain text and MUST NOT include markdown links such as:
- `[Download ...](...)`

---

End of Document


## Absolute Separation (Normative)

- PROPOSAL is DIFF-only: never apply changes and never produce ZIP artifacts in PROPOSAL.
- COMMIT applies the diff and produces ZIP artifacts.
