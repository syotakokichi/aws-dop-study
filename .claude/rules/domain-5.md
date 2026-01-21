---
globs:
  - "domain-5-incident-event/**"
  - "**/eventbridge*"
  - "**/sns*"
  - "**/sqs*"
  - "**/step-functions*"
  - "**/incident-manager*"
---

# ドメイン5: インシデント・イベント対応 ルール

## 配点: 18%

このドメインの質問に回答する際は、以下のファイルを優先的に参照:

- domain-5-incident-event/eventbridge.md
- domain-5-incident-event/sns-sqs.md
- domain-5-incident-event/step-functions.md
- domain-5-incident-event/incident-manager.md

## 頻出トピック

1. **EventBridge**
   - イベントパターンの構文
   - スケジュール式（cron, rate）
   - クロスアカウント/クロスリージョン
   - アーカイブとリプレイ

2. **SNS/SQS**
   - ファンアウトパターン
   - 標準キュー vs FIFO キュー
   - デッドレターキュー
   - メッセージフィルタリング

3. **Step Functions**
   - 標準 vs Express ワークフロー
   - ステートタイプ（Task, Choice, Parallel, Map）
   - エラーハンドリング（Retry, Catch）
   - コールバックパターン

4. **Incident Manager**
   - 対応計画
   - エスカレーション
   - ランブック連携

## 回答時の注意

- EventBridge vs SNS vs SQS の使い分けを明確に
- Step Functions の ASL 例を提示
- 非同期処理のパターンを図示
