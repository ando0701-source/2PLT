# 2PLT_00_ENTRYPOINT (Normative)

## Purpose
Define the deterministic entry procedure for an activated WORKER to interpret a MANAGER block,
without document hunting (no discovery; no “look around and decide”).

## Activation boundary (Normative)
This procedure applies only when a parsable `BEGIN_MANAGER` … `END_MANAGER` block exists.

## Deterministic entry procedure (Normative)
When activated, the WORKER MUST follow this procedure in order:

1. Read DOC_ID `2PLT_00_ENTRYPOINT` (this document).

2. Read the profile document indicated by `PROFILE_DOC_ID:` (DOC_ID `<PROFILE_DOC_ID>`).

3. If a payload line `DOC_SET: <DOC_SET_NAME>` exists:
   - Read DOC_ID `2PLT_20_DOC_SET_VOCAB`,
   - Resolve `<DOC_SET_NAME>` to an allowed DOC_ID set.

4. Consult ONLY:
   - the active profile DOC_ID, and
   - the resolved DOC_ID set (if any),
   for all normative interpretation.

## Bootstrap note (Normative)
Loading a document by DOC_ID is performed via a resolver initialized by the BOOT LOADER manifest,
as defined by DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE`.

## Prohibited discovery (Normative)
During activated processing, the WORKER MUST NOT:
- enumerate the container/artifact to discover documents, or
- consult any navigation/inventory/index/table-of-contents document to decide what to read next.

Any such behavior is a document governance violation and MUST be handled as defined by DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE`.

## Artifact channel invariants (Normative)
Unless a profile explicitly overrides:
- If `STATE=PROPOSAL` → `ARTIFACT=INLINE`
- If `STATE=ABEND` → `ARTIFACT=INLINE`
- If `STATE=COMMIT` → `ARTIFACT` references a produced artifact id (as defined by the active profile)
- If `STATE=UNRESOLVED` → `ARTIFACT` references a produced artifact id (as defined by the active profile)

Additionally:
- A PROPOSAL MUST NOT apply any patch; it proposes a mechanically-applicable diff/patch inline.
- A PROPOSAL MUST NOT produce any artifact file.

## Output format ban (Normative)
The WORKER response envelope MUST be plain text and MUST NOT include markdown links.

## Absolute separation (Normative)
- PROPOSAL is DIFF-only: never apply changes and never produce artifacts in PROPOSAL.
- COMMIT applies the diff and produces artifacts.

End of document.
