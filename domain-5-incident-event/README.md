# ドメイン5: インシデント・イベント対応 (18%)

## 概要

イベント駆動アーキテクチャとインシデント管理に関するドメイン。
EventBridge、自動化ワークフロー、障害対応が中心。

## 主要サービス

| サービス | 役割 |
|----------|------|
| EventBridge | イベントバス・ルール |
| SNS | プッシュ通知 |
| SQS | メッセージキュー |
| Lambda | イベントハンドラ |
| Step Functions | ワークフローオーケストレーション |
| Incident Manager | インシデント管理 |
| Health Dashboard | AWSサービス状態 |

## 試験頻出トピック

### EventBridge
- [ ] イベントパターンマッチング
- [ ] ルールとターゲット
- [ ] スケジュール（cron/rate）
- [ ] クロスアカウント・リージョン
- [ ] アーカイブとリプレイ
- [ ] スキーマレジストリ

### Step Functions
- [ ] Standard vs Express
- [ ] ASL（Amazon States Language）
- [ ] タスク状態・選択状態・並列状態
- [ ] エラーハンドリング（Retry, Catch）
- [ ] アクティビティ

### SNS/SQS
- [ ] ファンアウトパターン
- [ ] Dead Letter Queue
- [ ] メッセージフィルタリング
- [ ] FIFO vs Standard

### Incident Manager
- [ ] レスポンスプラン
- [ ] エスカレーション
- [ ] ランブック統合
- [ ] ポストインシデント分析

### 自動化シナリオ
- [ ] EC2インスタンス異常時の自動復旧
- [ ] セキュリティグループ変更の検知と修復
- [ ] コスト異常時のアラート

## 学習ファイル

- [eventbridge.md](./eventbridge.md)
- [sns-sqs.md](./sns-sqs.md)
- [step-functions.md](./step-functions.md)
- [incident-manager.md](./incident-manager.md)

## シナリオ別対応パターン

- [scenarios/](./scenarios/) - 典型的なシナリオと対応例

## ハンズオン

- [hands-on/](./hands-on/) - 実践記録
