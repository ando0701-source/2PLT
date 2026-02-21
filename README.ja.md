# 2PLT — Two-Phase Layered Trigger

2PLT（Two-Phase Layered Trigger）は、処理の起動を「トリガ」で統一し、運用を **二段階（PROPOSAL → COMMIT）** に分離しつつ、例外系も **UNRESOLVED / ABEND** として明示的に扱うための枠組みです。

開始地点: [`docs/2PLT_00_ENTRYPOINT.md`](docs/2PLT_00_ENTRYPOINT.md)  
English: [`README.md`](README.md)

## 目的
- 明示的なトリガによる予測可能な実行
- 二段階プロセスによる監査可能・決定的な変更管理
- 人手介入や失敗を終端状態として明確化

## 基本概念
- **Phase**：PROPOSAL → COMMIT
- **終端状態**：UNRESOLVED / ABEND
- **トリガ駆動**：例 `@@@@2PLT_...@@@@`

## 読む順（最短）
- `docs/2PLT_00_ENTRYPOINT.md` — 起動時の決定的手順（ZIP全走査禁止）
- `docs/INDEX.md` — “次に読む” ルート

## 状態
本リポジトリは実験・検証目的で運用しています。

## "REJECT" について
リポジトリの説明文などに `REJECT` が出る場合がありますが、現状このリポジトリの規範（Normative）文書では終端状態は `UNRESOLVED` と `ABEND` を採用しています。`REJECT` を導入する場合は、必ず profile / doc-set により明示的に定義してください。
