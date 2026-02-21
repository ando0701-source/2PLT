# 2PLT — Two-Phase Layered Trigger

2PLT（Two-Phase Layered Trigger）は、処理の起動を「トリガ」で統一し、運用を **二段階（PROPOSAL → COMMIT）** に分離しつつ、例外系も **UNRESOLVED / ABEND** として明示的に扱うための枠組みです。

## 目的
- 明示的なトリガによる予測可能な実行
- 二段階プロセスによる監査可能・決定的な変更管理
- 人手介入や失敗を終端状態として明確化

## 基本概念
- **Phase**：PROPOSAL → COMMIT
- **終端状態**：UNRESOLVED / ABEND
- **トリガ駆動**：例 `@@@@2PLT_...@@@@`

## 状態
本リポジトリは実験・検証目的で運用しています。