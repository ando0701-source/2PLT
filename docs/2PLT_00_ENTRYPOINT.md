# 2PLT_00_ENTRYPOINT (Normative)

## Purpose
Define the deterministic entry procedure for an activated WORKER to interpret a MANAGER block,
without document hunting (no discovery; no “look around and decide”).

## BOOT LOADER Selection (Normative)

At boot time (before vocabulary-based identification is available), the WORKER MUST select exactly one BOOT LOADER JSON by the trigger token:

- `@@@@2PLT_JL_PROPOSAL@@@@` → `docs/2PLT_00_ENTRYPOINT_PROPOSAL.json`
- `@@@@2PLT_JL_COMMIT@@@@` → `docs/2PLT_00_ENTRYPOINT_COMMIT.json`
- `@@@@2PLT_JL_REJECT@@@@` → `docs/2PLT_00_ENTRYPOINT_REJECT.json`

The selected BOOT LOADER JSON provides a physical-path-only load order.
The WORKER MUST load each file in the listed order and MUST NOT enumerate the repository or artifact to discover additional documents.

## Resolver initialization (Normative)

After BOOT loading completes, the WORKER initializes logical identification using the Document ID Vocabulary:

- Use the Document ID Vocabulary (DOC_ID `2PLT_05_DOC_ID_VOCAB`), which is already included in the BOOT load order.
- Treat **DOC_ID** as the only cross-document reference form inside governance/spec documents.
- Any further document access MUST remain within the BOOT-loaded set (no additional file discovery).

## Activation boundary (Normative)
This procedure applies only when a parsable `BEGIN_MANAGER` … `END_MANAGER` block exists.

## Deterministic entry procedure (Normative)
When activated, the WORKER MUST follow this procedure in order:

1. Read this document (DOC_ID `2PLT_00_ENTRYPOINT`).

2. Read the profile document indicated by `PROFILE_DOC_ID:` (DOC_ID `<PROFILE_DOC_ID>`).

3. Consult only:
   - the active profile document, and
   - documents already included in the BOOT-loaded set,
   for all normative interpretation.

## Prohibited discovery (Normative)
During activated processing, the WORKER MUST NOT:
- enumerate the container/repository to discover documents, or
- consult any navigation/inventory/index/table-of-contents document to decide what to read next.

Any such behavior is a document governance violation.

End of document.
