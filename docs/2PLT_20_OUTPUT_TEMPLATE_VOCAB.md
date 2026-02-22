# 2PLT Output Template Vocabulary (Normative)

## 1. Purpose
This document defines canonical `OUTPUT_TEMPLATE_ID` values and their exact **output templates**.
It exists to:
- reduce MANAGER prompt size (MANAGER references a template id),
- make WORKER output formatting mechanically verifiable,
- eliminate ad-hoc markdown formatting (bullets, indentation) in terminal envelopes.

## 2. Rules (Normative)

1. If MANAGER payload contains `OUTPUT_TEMPLATE_ID: <id>`, the WORKER MUST:
   - select the matching template in this document,
   - emit the template **exactly** (same keys, same order, same punctuation),
   - replace placeholders only (strings inside `<...>`),
   - emit **no extra lines** and no wrapper tokens (`BEGIN_*` / `END_*`).

2. List markers:
   - Only `- ` (hyphen + space) is permitted for list items.
   - Markdown bullets like `* ` are forbidden.

3. Diff block:
   - `PROPOSED_DIFF:` begins a raw unified-diff block.
   - The unified diff lines MUST start at column 1 (no indentation).
   - The diff block MUST continue until end-of-output.

## 3. Vocabulary

### JL_PROPOSAL_ENVELOPE_V1

Template (WORKER MUST output exactly this structure):

STATE: PROPOSAL
ARTIFACT: INLINE
TRIGGER: @@@@2PLT_JL_PROPOSAL@@@@
OWNER_ID: <OWNER_ID>
LANE_ID: <LANE_ID>
REQUEST_ID: <REQUEST_ID>
IN_STATE: <IN_STATE>
OUT_STATE: PROPOSAL
ARTIFACT_CLASS: LOGICAL
ARTIFACT_FORMAT: INLINE
NOTES:
- doc_set: <DOC_SET> (optional label; not used for BOOT discovery)
- proposal_mode: DIFF_ONLY
- proposal_input_zip: <proposal_input_zip>
- proposal_target: <proposal_target>
- proposed_diff: unified
PROPOSED_DIFF:
<PUT UNIFIED DIFF HERE>
