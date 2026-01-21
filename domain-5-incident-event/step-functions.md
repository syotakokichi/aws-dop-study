# AWS Step Functions

## 概要

AWS Step Functionsは、視覚的なワークフローを使用して分散アプリケーションを構築するサーバーレスオーケストレーションサービス。複数のAWSサービスを組み合わせた複雑なワークフローを、状態マシン（ステートマシン）として定義・実行・管理できる。

## キーコンセプト

- **ステートマシン（State Machine）**: ワークフローの定義
- **ステート（State）**: ワークフロー内の個々の状態/タスク
- **実行（Execution）**: ステートマシンの1回の実行インスタンス
- **ASL（Amazon States Language）**: ワークフローを定義するJSON形式の言語
- **標準ワークフロー**: 長時間実行、高耐久性
- **Express ワークフロー**: 短時間、高スループット、低コスト
- **コールバックパターン**: 外部処理の完了を待機
- **動的並列処理**: データセットの各要素を並列処理

## 試験頻出ポイント

- [ ] ステートタイプ（Task, Choice, Parallel, Map, Wait, Pass, Succeed, Fail）
- [ ] 標準 vs Express ワークフローの違いと選択基準
- [ ] エラーハンドリング（Retry, Catch）
- [ ] サービス統合パターン（Request Response, Run a Job, Wait for Callback）
- [ ] 動的並列処理（Map state）
- [ ] 入出力処理（InputPath, OutputPath, ResultPath, Parameters, ResultSelector）
- [ ] コールバックパターン（タスクトークン）
- [ ] EventBridgeとの統合

## DOP-C02での出題パターン

### パターン1: 長時間ワークフロー
- **シナリオ**: データ処理パイプラインで複数のジョブを順次実行
- **ポイント**: 標準ワークフロー + Run a Job統合

### パターン2: 高スループット処理
- **シナリオ**: IoTデータをリアルタイムで処理
- **ポイント**: Express ワークフロー

### パターン3: エラー時のリトライと分岐
- **シナリオ**: API呼び出し失敗時にリトライし、それでも失敗したら通知
- **ポイント**: Retry + Catch + SNS

### パターン4: 人間の承認待ち
- **シナリオ**: ワークフロー途中で人間の承認を待機
- **ポイント**: コールバックパターン（Wait for Callback）

### パターン5: 並列処理
- **シナリオ**: 大量のファイルを並列で処理
- **ポイント**: Map state（動的並列処理）

## ベストプラクティス

1. **適切なワークフロータイプの選択**: 実行時間とスループット要件で判断
2. **エラーハンドリングの実装**: Retry + Catchで堅牢なワークフロー
3. **べき等性の確保**: 再試行時に安全に実行できるよう設計
4. **入出力処理の最適化**: 必要なデータのみ受け渡し
5. **ログの有効化**: CloudWatch Logsでデバッグを容易に
6. **X-Rayトレーシング**: 分散トレースで問題を特定

## ステートタイプ

| タイプ | 説明 | ユースケース |
|--------|------|--------------|
| Task | AWSサービスまたはアクティビティを実行 | Lambda呼び出し、Batch実行 |
| Choice | 条件分岐 | if-else ロジック |
| Parallel | 複数ブランチを並列実行 | 独立した処理の同時実行 |
| Map | 配列の各要素を処理 | バッチ処理、ETL |
| Wait | 指定時間または時刻まで待機 | 遅延実行、スケジュール |
| Pass | 入力を出力にパススルー | デバッグ、データ変換 |
| Succeed | 成功で終了 | 正常終了 |
| Fail | 失敗で終了 | エラー終了 |

## 標準 vs Express ワークフロー

| 項目 | 標準 | Express |
|------|------|---------|
| 最大実行時間 | 1年 | 5分 |
| スループット | 2,000/秒 | 100,000/秒 |
| 実行履歴 | 90日保持 | CloudWatch Logsに送信 |
| 実行セマンティクス | exactly-once | at-least-once (同期), at-most-once (非同期) |
| 料金 | 状態遷移ごと | 実行回数と時間 |
| ユースケース | 長時間処理、オーケストレーション | イベント処理、IoT |

## ASL（Amazon States Language）例

```json
{
  "Comment": "注文処理ワークフロー",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:validateOrder",
      "Next": "CheckInventory",
      "Retry": [
        {
          "ErrorEquals": ["ServiceException"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["ValidationError"],
          "Next": "OrderFailed"
        }
      ]
    },
    "CheckInventory": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.inStock",
          "BooleanEquals": true,
          "Next": "ProcessPayment"
        }
      ],
      "Default": "BackorderNotification"
    },
    "ProcessPayment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:processPayment",
      "Next": "ShipOrder"
    },
    "ShipOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:shipOrder",
      "End": true
    },
    "BackorderNotification": {
      "Type": "Task",
      "Resource": "arn:aws:sns:...:backorder-topic",
      "End": true
    },
    "OrderFailed": {
      "Type": "Fail",
      "Cause": "Order validation failed"
    }
  }
}
```

## サービス統合パターン

| パターン | 説明 | ユースケース |
|----------|------|--------------|
| Request Response | 即時レスポンス | Lambda同期呼び出し |
| Run a Job (.sync) | ジョブ完了まで待機 | Batch, ECS Task, Glue |
| Wait for Callback (.waitForTaskToken) | 外部からの完了通知を待機 | 人間の承認、外部システム |

### コールバックパターン例

```json
{
  "WaitForApproval": {
    "Type": "Task",
    "Resource": "arn:aws:states:::sqs:sendMessage.waitForTaskToken",
    "Parameters": {
      "QueueUrl": "https://sqs...",
      "MessageBody": {
        "TaskToken.$": "$$.Task.Token",
        "OrderId.$": "$.orderId"
      }
    },
    "Next": "ProcessApproval"
  }
}
```

### コールバック送信（外部から）

```python
import boto3

sfn = boto3.client('stepfunctions')

# 成功を通知
sfn.send_task_success(
    taskToken='TOKEN_FROM_MESSAGE',
    output='{"approved": true}'
)

# 失敗を通知
sfn.send_task_failure(
    taskToken='TOKEN_FROM_MESSAGE',
    error='ApprovalDenied',
    cause='Manager rejected the request'
)
```

## 入出力処理

| フィールド | 説明 | 適用タイミング |
|------------|------|----------------|
| InputPath | 入力のどの部分を使用するか | 最初 |
| Parameters | 入力データを変換 | InputPath後 |
| ResultSelector | タスク結果を変換 | タスク完了後 |
| ResultPath | 結果をどこに配置するか | ResultSelector後 |
| OutputPath | 出力のどの部分を次に渡すか | 最後 |

```json
{
  "Type": "Task",
  "InputPath": "$.order",
  "Parameters": {
    "orderId.$": "$.id",
    "staticValue": "fixed"
  },
  "ResultSelector": {
    "status.$": "$.Payload.status"
  },
  "ResultPath": "$.taskResult",
  "OutputPath": "$"
}
```

## Map State（動的並列処理）

```json
{
  "ProcessItems": {
    "Type": "Map",
    "ItemsPath": "$.items",
    "MaxConcurrency": 10,
    "ItemProcessor": {
      "ProcessorConfig": {
        "Mode": "INLINE"
      },
      "StartAt": "ProcessItem",
      "States": {
        "ProcessItem": {
          "Type": "Task",
          "Resource": "arn:aws:lambda:...:processItem",
          "End": true
        }
      }
    },
    "End": true
  }
}
```

## 他サービスとの連携

| サービス | 統合タイプ |
|----------|------------|
| Lambda | 最適化統合（直接呼び出し） |
| ECS/Fargate | Run a Job |
| Batch | Run a Job |
| Glue | Run a Job |
| EMR | Run a Job |
| SageMaker | Run a Job |
| SNS | Request Response |
| SQS | Request Response / Callback |
| DynamoDB | Request Response |
| EventBridge | Request Response |
| API Gateway | Request Response |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS Step Functions 開発者ガイド](https://docs.aws.amazon.com/step-functions/latest/dg/)
- [Amazon States Language](https://states-language.net/spec.html)
- [サービス統合](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-service-integrations.html)
