# 2PLT_00_ENTRYPOINT (Normative)

## Purpose
Define the deterministic entry procedure for an activated WORKER to interpret a MANAGER block,
without document hunting (no discovery; no “look around and decide”).

## Resolver initialization (Normative)
Before loading any documents by DOC_ID, the implementation MUST initialize a resolver as defined by
DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE`:

1. Load BOOT LOADER (`2PLT_00_ENTRYPOINT.json`) from its fixed location.
2. Load DOC_VOCAB (DOC_ID `2PLT_05_DOC_ID_VOCAB`) from its fixed location.
3. Build a resolver using:
   - DOC_VOCAB mapping: DOC_ID → filename
   - BOOT LOADER paths: docs_root_relpath + filename → physical_relpath
4. Enforce: governance/spec documents are loaded by DOC_ID only (no filename references in docs).

This resolver initialization MUST NOT involve discovery.

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

## Inspection reading order (Informative)
For document inspection (lint), the typical first document is DOC_ID `2PLT_05_DOC_ID_VOCAB`
to establish the DOC_ID universe.

## Prohibited discovery (Normative)
During activated processing, the WORKER MUST NOT:
- enumerate the container/repository to discover documents, or
- consult any navigation/inventory/index/table-of-contents document to decide what to read next.

Any such behavior is a document governance violation.

End of document.
