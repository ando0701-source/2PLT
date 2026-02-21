# 2PLT — Two-Phase Layered Trigger

2PLT (Two-Phase Layered Trigger) is a layered trigger framework with a two-phase flow (PROPOSAL → COMMIT) plus UNRESOLVED / ABEND.

## Goals
- Predictable execution via explicit triggers
- Auditable, deterministic changes through a two-phase process
- Clear terminal states for exceptions and human intervention

## Core Concepts
- **Phase**: PROPOSAL → COMMIT
- **Terminal states**: UNRESOLVED / ABEND
- **Trigger-driven**: e.g., `@@@@2PLT_...@@@@`

## Status
This repository is currently used for experiments and verification.