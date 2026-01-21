# Amazon SNS & Amazon SQS

## 概要

### Amazon SNS (Simple Notification Service)
フルマネージドのPub/Subメッセージングサービス。トピックへのメッセージを複数のサブスクライバーにファンアウト配信する。プッシュベースの配信モデル。

### Amazon SQS (Simple Queue Service)
フルマネージドのメッセージキューイングサービス。プロデューサーとコンシューマー間の非同期通信を実現。プルベースの配信モデル。

## キーコンセプト

### SNS
- **トピック**: メッセージの配信先（論理アクセスポイント）
- **サブスクリプション**: トピックのメッセージ受信設定
- **メッセージフィルタリング**: 属性に基づいてサブスクライバーをフィルタリング
- **ファンアウト**: 1メッセージを複数宛先に配信
- **FIFO トピック**: メッセージの順序保証

### SQS
- **キュー**: メッセージを保存するバッファ
- **標準キュー**: 高スループット、順序保証なし（at-least-once配信）
- **FIFOキュー**: 厳密な順序保証（exactly-once処理）
- **可視性タイムアウト**: 処理中のメッセージを他のコンシューマーから隠す
- **デッドレターキュー（DLQ）**: 処理失敗メッセージの退避先
- **ロングポーリング**: 空のレスポンスを減らしコストを削減

## 試験頻出ポイント

### SNS
- [ ] サポートされるプロトコル（HTTP/S, Email, SMS, Lambda, SQS, Kinesis Firehose）
- [ ] メッセージフィルタリングポリシー
- [ ] ファンアウトパターン（SNS + 複数SQS）
- [ ] FIFO vs 標準トピック
- [ ] クロスアカウント/クロスリージョン配信
- [ ] 配信再試行ポリシー

### SQS
- [ ] 標準キュー vs FIFOキュー
- [ ] 可視性タイムアウトの設計
- [ ] デッドレターキューの設定
- [ ] ロングポーリング vs ショートポーリング
- [ ] メッセージ保持期間（最大14日）
- [ ] 遅延キュー（Delay Queue）

## DOP-C02での出題パターン

### パターン1: デカップリング
- **シナリオ**: マイクロサービス間の疎結合な通信
- **ポイント**: SQSキューによるバッファリング

### パターン2: ファンアウト
- **シナリオ**: 1つのイベントを複数のサービスで処理
- **ポイント**: SNSトピック + 複数SQSサブスクリプション

### パターン3: 順序保証
- **シナリオ**: メッセージの処理順序を保証したい
- **ポイント**: FIFOキュー、メッセージグループID

### パターン4: 失敗メッセージの処理
- **シナリオ**: 処理失敗したメッセージを分析・再処理
- **ポイント**: デッドレターキュー + リドライブポリシー

### パターン5: 通知
- **シナリオ**: CloudWatchアラームの通知先
- **ポイント**: SNSトピック → メール/Slack/PagerDuty

## ベストプラクティス

### SNS
1. **メッセージフィルタリングの活用**: サブスクライバー側での不要な処理を削減
2. **配信再試行の設定**: エンドポイント障害に対応
3. **暗号化の有効化**: SSE-KMSでメッセージを暗号化
4. **アクセスポリシーの最小権限化**: 必要なプリンシパルのみ許可

### SQS
1. **可視性タイムアウトの適切な設定**: 処理時間 + バッファ
2. **DLQの設定**: 処理失敗メッセージの追跡
3. **ロングポーリングの使用**: コスト削減とレイテンシ改善
4. **バッチ処理の活用**: API呼び出し回数を削減

## 標準キュー vs FIFOキュー

| 項目 | 標準キュー | FIFOキュー |
|------|------------|------------|
| スループット | 無制限 | 3,000 msg/秒（バッチ） |
| 順序保証 | ベストエフォート | 厳密な順序保証 |
| 配信保証 | at-least-once | exactly-once |
| 重複 | 可能性あり | なし |
| キュー名 | 任意 | `.fifo`サフィックス必須 |
| ユースケース | 高スループット | 金融取引、順序重要な処理 |

## SNS + SQS ファンアウトパターン

```
                ┌─→ SQS Queue A → Lambda A
                │
SNS Topic ──────┼─→ SQS Queue B → Lambda B
                │
                └─→ SQS Queue C → ECS Service
```

### 設定例

```yaml
# SNS トピック
MyTopic:
  Type: AWS::SNS::Topic
  Properties:
    TopicName: my-topic

# SQS キュー
MyQueue:
  Type: AWS::SQS::Queue
  Properties:
    QueueName: my-queue
    RedrivePolicy:
      deadLetterTargetArn: !GetAtt MyDLQ.Arn
      maxReceiveCount: 3

# SNS → SQS サブスクリプション
MySubscription:
  Type: AWS::SNS::Subscription
  Properties:
    TopicArn: !Ref MyTopic
    Protocol: sqs
    Endpoint: !GetAtt MyQueue.Arn
    FilterPolicy:
      eventType:
        - order_created
        - order_updated
```

## メッセージフィルタリング（SNS）

```json
// フィルタリングポリシー
{
  "eventType": ["order_created"],
  "store": [{"prefix": "store-"}],
  "price": [{"numeric": [">=", 100]}],
  "isPrime": [true]
}

// メッセージ属性
{
  "eventType": {"DataType": "String", "StringValue": "order_created"},
  "store": {"DataType": "String", "StringValue": "store-tokyo"},
  "price": {"DataType": "Number", "StringValue": "150"},
  "isPrime": {"DataType": "String", "StringValue": "true"}
}
```

## 可視性タイムアウト

```
1. コンシューマーがメッセージを受信
2. 可視性タイムアウト期間中、他のコンシューマーには見えない
3. 処理完了 → メッセージを削除
4. 処理失敗 → タイムアウト後に再度可視化 → リトライ
5. 最大受信回数超過 → DLQに移動
```

### 推奨設定

```
可視性タイムアウト = 処理時間の6倍 または 最大処理時間 + バッファ
```

## デッドレターキュー（DLQ）

```yaml
# DLQの設定
MainQueue:
  Type: AWS::SQS::Queue
  Properties:
    RedrivePolicy:
      deadLetterTargetArn: !GetAtt DLQ.Arn
      maxReceiveCount: 3  # 3回失敗でDLQへ

DLQ:
  Type: AWS::SQS::Queue
  Properties:
    MessageRetentionPeriod: 1209600  # 14日
```

### リドライブ（DLQから元キューへ）

```bash
aws sqs start-message-move-task \
  --source-arn arn:aws:sqs:region:account:my-dlq \
  --destination-arn arn:aws:sqs:region:account:my-queue
```

## SNS vs SQS vs EventBridge

| 項目 | SNS | SQS | EventBridge |
|------|-----|-----|-------------|
| パターン | Pub/Sub | Point-to-Point | Event Router |
| 配信 | プッシュ | プル | プッシュ |
| 永続化 | なし | あり（14日） | アーカイブ可能 |
| フィルタリング | 属性ベース | なし | 高度なパターンマッチ |
| ユースケース | ファンアウト、通知 | デカップリング、バッファ | イベント駆動、ルーティング |

## 他サービスとの連携

### SNS

| サービス | 連携内容 |
|----------|----------|
| SQS | ファンアウトパターン |
| Lambda | サーバーレス処理 |
| HTTP/HTTPS | Webhook |
| Email/SMS | 通知 |
| CloudWatch | アラーム通知 |
| Kinesis Firehose | ストリーミング配信 |

### SQS

| サービス | 連携内容 |
|----------|----------|
| Lambda | イベントソースマッピング |
| ECS | タスクスケーリング |
| EC2 Auto Scaling | メッセージ数ベーススケーリング |
| Step Functions | ワークフロー統合 |
| SNS | ファンアウト受信 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [Amazon SNS ユーザーガイド](https://docs.aws.amazon.com/sns/latest/dg/)
- [Amazon SQS ユーザーガイド](https://docs.aws.amazon.com/sqs/latest/dg/)
- [SNS メッセージフィルタリング](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html)
- [SQS デッドレターキュー](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
