# 00_AI_BOOTSTRAP_MANUAL (Informative)

## 1. Purpose

This document describes the **pre-activation BOOTSTRAP handshake** used to deliver the canonical 2PLT ruleset ZIP
to the **WORKER execution endpoint** before any activated `BEGIN_MANAGER ... END_MANAGER` request is sent.

This handshake is **non-activated** and is governed normatively by:
- DOC_ID `2PLT_00_DOCUMENT_GOVERNANCE` (section 11.4)
- DOC_ID `2PLT_10_STATE_MACHINE` (section 3.1)

## 2. Role Determinism

- In a WORKER thread, the receiver MUST assume it is operating as **WORKER**.
- The receiver MUST NOT narrate or assume a "manager" / "administrator" role during the BOOTSTRAP handshake.
- The BOOTSTRAP handshake does not select a 2PLT profile and does not activate 2PLT.

## 3. Input Format (Strict)

A BOOTSTRAP handshake message MUST contain **only** the following five lines (field order REQUIRED):

```
BOOTSTRAP_HANDSHAKE
BOOTSTRAP_TARGET: WORKER
BOOTSTRAP_CONTEXT: CLEAN
BOOTSTRAP_ZIP: <input_zip_filename>
BOOTSTRAP_OUTPUT: ACK_ONLY
```

No additional non-empty lines are permitted (including separators like `-----`).

## 4. Output Tokens (Strict)

In BOOTSTRAP_HANDSHAKE_MODE, the WORKER output MUST be exactly one of:
- `ACK`
- `NACK`
- `INPUT_MISSING`

and MUST be the entire output (single line, no other text, no links, no citations).

After emitting the token, the WORKER MUST STOP immediately and MUST NOT add any additional lines.

Deterministic selection rules are defined by DOC_ID `2PLT_10_STATE_MACHINE` (section 3.1).

## 5. Context / Retrieval Rules (Strict)

During the BOOTSTRAP handshake, the WORKER MUST:
- NOT retrieve or reference any past chats, memories, external sources, or citations.
- NOT output any Markdown links or footnotes.

## 6. ZIP Handling Rules (Strict)

During the BOOTSTRAP handshake, the WORKER MUST:
- NOT open, list, extract, or semantically validate the ZIP archive (BOOTSTRAP is role pinning only).
- NOT inspect or interpret triggers.

## 7. Completion

After receiving `ACK`, the sender may proceed to send the first activated MANAGER block.
Activation requires a `BEGIN_MANAGER` boundary token (DOC_ID `2PLT_10_STATE_MACHINE`).

---
End of Document
