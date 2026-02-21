# 2PLT_RESPONSIBILITY (Normative)

## 1. Responsibility Classes

2PLT distinguishes two responsibility classes:

- **Logical responsibility:** producing correct terminal state and compliant output structure.
- **Physical responsibility:** performing mutations to external artifacts (e.g., writing/updating logs, generating ZIP artifacts).

## 2. Terminal-to-Responsibility Mapping

### PROPOSAL
- Logical responsibility: REQUIRED
- Physical responsibility: PROHIBITED

### COMMIT
- Logical responsibility: REQUIRED
- Physical responsibility: REQUIRED (must produce/modify the specified artifact deterministically)

### UNRESOLVED
- Logical responsibility: REQUIRED (must report inability to finalize)
- Physical responsibility: REQUIRED (must produce/update the profile-defined unresolved artifact when required by the output schema)

### ABEND
- Logical responsibility: REQUIRED (must report failure)
- Physical responsibility: PROHIBITED

## 3. Prohibitions

- Physical mutations MUST NOT occur in PROPOSAL or ABEND.
- Physical mutations are permitted only for terminals that require external artifacts (COMMIT and UNRESOLVED).
- “Soft commits” (claiming COMMIT without producing the required artifact) are prohibited.

