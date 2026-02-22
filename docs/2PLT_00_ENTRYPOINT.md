# 2PLT_00_ENTRYPOINT (Normative)

## BOOT LOADER Selection (Normative)

At boot time (before DOC_ID resolution is available), the WORKER MUST select exactly one BOOT LOADER JSON by the trigger token:

- `@@@@2PLT_JL_PROPOSAL@@@@` → `docs/2PLT_00_ENTRYPOINT_PROPOSAL.json`
- `@@@@2PLT_JL_COMMIT@@@@` → `docs/2PLT_00_ENTRYPOINT_COMMIT.json`
- `@@@@2PLT_JL_REJECT@@@@` → `docs/2PLT_00_ENTRYPOINT_REJECT.json`

The selected BOOT LOADER JSON provides a physical-path-only load order. The WORKER MUST load each file in order.
The BOOT LOADER JSON MUST NOT contain DOC_ID-based rules.



## Purpose
Define the deterministic entry procedure for an activated WORKER to interpret a MANAGER block,
without document hunting (no discovery; no “look around and decide”).

## Resolver initialization (Normative)

BOOT stage does not require DOC_ID resolution.

1. Select exactly one BOOT LOADER JSON by the trigger token (see **BOOT LOADER Selection** above).
2. Load each file listed in the selected BOOT LOADER JSON **by physical path**, in the given order.
3. For interpretation after BOOT, use only the documents that were loaded in step (2). Do not discover or load additional documents.

## Activation boundary (Normative)
This procedure applies only when a parsable `BEGIN_MANAGER` … `END_MANAGER` block exists.

## Deterministic entry procedure (Normative)

When activated, the WORKER MUST follow this procedure in order:

1. BOOT: select the BOOT LOADER JSON by trigger and load the physical-path document set.
2. Interpret the MANAGER block using only the already-loaded documents.
3. Any attempt to consult documents outside the BOOT-loaded set is a governance violation.

## Prohibited discovery (Normative)
During activated processing, the WORKER MUST NOT:
- enumerate the container/repository to discover documents, or
- consult any navigation/inventory/index/table-of-contents document to decide what to read next.

Any such behavior is a document governance violation.

End of document.
