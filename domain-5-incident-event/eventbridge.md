# Amazon EventBridge

## 概要

Amazon EventBridgeは、AWSサービス、SaaSアプリケーション、カスタムアプリケーションからのイベントを使用して、アプリケーションを接続するサーバーレスイベントバスサービス。イベント駆動アーキテクチャの中核として、疎結合なシステム統合を実現する。

## キーコンセプト

- **イベントバス**: イベントを受信するチャネル（デフォルト、カスタム、パートナー）
- **ルール**: イベントパターンに基づいてターゲットにルーティング
- **イベントパターン**: イベントをフィルタリングする条件
- **ターゲット**: イベントを受信して処理するAWSサービス
- **スケジュール**: cron式またはrate式で定期実行
- **スキーマレジストリ**: イベントの構造を検出・保存
- **アーカイブとリプレイ**: イベントの保存と再実行
- **パイプ**: イベントソースとターゲットを直接接続（フィルタリング・強化付き）

## 試験頻出ポイント

- [ ] イベントパターンの構文（prefix, suffix, numeric, exists等）
- [ ] ターゲットの種類（Lambda, Step Functions, SNS, SQS, Kinesis等）
- [ ] スケジュール式（cron, rate）
- [ ] クロスアカウント/クロスリージョンのイベント配信
- [ ] デッドレターキュー（DLQ）の設定
- [ ] 入力トランスフォーマー
- [ ] アーカイブとリプレイ機能
- [ ] EventBridge Schedulerとの違い

## DOP-C02での出題パターン

### パターン1: AWSサービスイベントの処理
- **シナリオ**: EC2インスタンスの状態変更をトリガーに処理を実行
- **ポイント**: デフォルトイベントバス + EC2 State Changeイベント

### パターン2: 定期実行タスク
- **シナリオ**: 毎日深夜にLambda関数を実行
- **ポイント**: スケジュールルール（cron式）

### パターン3: クロスアカウントイベント
- **シナリオ**: 複数アカウントのイベントを中央アカウントで処理
- **ポイント**: イベントバスポリシー、ターゲットとしての別アカウントのイベントバス

### パターン4: イベントの再処理
- **シナリオ**: 過去のイベントを再処理したい
- **ポイント**: アーカイブとリプレイ機能

### パターン5: 失敗時の対応
- **シナリオ**: ターゲットへの配信失敗時にイベントを保存
- **ポイント**: デッドレターキュー（SQS）

## ベストプラクティス

1. **イベントパターンの最適化**: 必要なイベントのみフィルタリング
2. **DLQの設定**: ターゲット障害時のイベント損失を防止
3. **入力トランスフォーマーの活用**: ターゲットに必要な形式に変換
4. **アーカイブの活用**: 監査や再処理のためにイベントを保存
5. **スキーマレジストリの活用**: イベント構造のドキュメント化
6. **リトライポリシーの設定**: 一時的な障害に対応

## イベントパターン

### 基本構文

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running", "stopped"]
  }
}
```

### 高度なパターンマッチング

```json
{
  "detail": {
    // 前方一致
    "key": [{"prefix": "prod-"}],

    // 後方一致
    "filename": [{"suffix": ".json"}],

    // 数値比較
    "count": [{"numeric": [">=", 10, "<", 100]}],

    // 存在チェック
    "optionalField": [{"exists": true}],

    // 否定
    "status": [{"anything-but": ["failed"]}],

    // 空でない
    "data": [{"exists": true}],

    // OR条件
    "region": ["ap-northeast-1", "us-east-1"],

    // ワイルドカード
    "name": [{"wildcard": "prod-*-app"}]
  }
}
```

## ターゲット

### サポートされるターゲット

| カテゴリ | ターゲット |
|----------|------------|
| コンピューティング | Lambda, ECS Task, Step Functions |
| メッセージング | SNS, SQS, Kinesis Data Streams, Kinesis Firehose |
| オートメーション | Systems Manager Automation, CodePipeline, CodeBuild |
| 別イベントバス | 同一/別アカウント/別リージョンのEventBridge |
| API | API Gateway, API Destinations (外部API) |
| その他 | Batch, Incident Manager, Redshift |

### ターゲット設定例

```yaml
# CloudFormation
EventRule:
  Type: AWS::Events::Rule
  Properties:
    EventBusName: default
    EventPattern:
      source:
        - aws.ec2
      detail-type:
        - EC2 Instance State-change Notification
    Targets:
      - Id: LambdaTarget
        Arn: !GetAtt MyLambda.Arn
        DeadLetterConfig:
          Arn: !GetAtt MyDLQ.Arn
        RetryPolicy:
          MaximumRetryAttempts: 3
          MaximumEventAgeInSeconds: 3600
```

## スケジュール

### cron式

```
cron(分 時 日 月 曜日 年)

# 毎日9:00 UTC
cron(0 9 * * ? *)

# 平日の18:00 JST (9:00 UTC)
cron(0 9 ? * MON-FRI *)

# 毎月1日の0:00
cron(0 0 1 * ? *)
```

### rate式

```
rate(5 minutes)
rate(1 hour)
rate(1 day)
```

## 入力トランスフォーマー

```yaml
InputTransformer:
  InputPathsMap:
    instance: $.detail.instance-id
    state: $.detail.state
    time: $.time
  InputTemplate: |
    {
      "message": "Instance <instance> changed to <state> at <time>"
    }
```

## クロスアカウント/クロスリージョン

### イベントバスポリシー（受信側）

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCrossAccountPutEvents",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::SOURCE_ACCOUNT:root"
      },
      "Action": "events:PutEvents",
      "Resource": "arn:aws:events:REGION:DEST_ACCOUNT:event-bus/central-bus"
    }
  ]
}
```

### 送信側ルール設定

```yaml
# ターゲットとして別アカウントのイベントバスを指定
Targets:
  - Id: CrossAccountTarget
    Arn: arn:aws:events:REGION:DEST_ACCOUNT:event-bus/central-bus
    RoleArn: !GetAtt CrossAccountRole.Arn
```

## アーカイブとリプレイ

### アーカイブ作成

```yaml
EventArchive:
  Type: AWS::Events::Archive
  Properties:
    SourceArn: !GetAtt MyEventBus.Arn
    EventPattern:
      source:
        - myapp
    RetentionDays: 30
```

### リプレイ実行

```bash
aws events start-replay \
  --event-source-arn arn:aws:events:region:account:archive/my-archive \
  --destination arn:aws:events:region:account:event-bus/default \
  --event-start-time 2024-01-01T00:00:00Z \
  --event-end-time 2024-01-02T00:00:00Z
```

## EventBridge Scheduler

| 項目 | EventBridge Rules | EventBridge Scheduler |
|------|-------------------|----------------------|
| スケジュール数 | 300/バス | 100万+ |
| 一度限りの実行 | ❌ | ✓ |
| タイムゾーン指定 | ❌ | ✓ |
| フレキシブルタイムウィンドウ | ❌ | ✓ |
| ユースケース | イベントルーティング | 大規模スケジュール |

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| Lambda | 最も一般的なターゲット |
| Step Functions | ワークフロー起動 |
| SNS/SQS | メッセージング連携 |
| CodePipeline | パイプライン起動 |
| Systems Manager | 自動化アクション |
| CloudWatch | イベントメトリクス |
| API Gateway | HTTP APIトリガー |
| Incident Manager | インシデント作成 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [Amazon EventBridge ユーザーガイド](https://docs.aws.amazon.com/eventbridge/latest/userguide/)
- [イベントパターン](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)
- [EventBridge Scheduler](https://docs.aws.amazon.com/scheduler/latest/UserGuide/)
