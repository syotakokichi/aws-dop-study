# Amazon CloudWatch Logs

## 概要

Amazon CloudWatch Logsは、アプリケーション、システム、AWSサービスからのログデータを収集、監視、保存、分析するためのサービス。リアルタイムのログ監視、長期保存、ログデータに基づくメトリクス生成、他のAWSサービスへのログ配信が可能。

## キーコンセプト

- **ロググループ（Log Group）**: ログストリームをグループ化するコンテナ
- **ログストリーム（Log Stream）**: 同一ソースからのログイベントの集合
- **ログイベント**: タイムスタンプとメッセージを持つ個々のログレコード
- **メトリクスフィルター**: ログからカスタムメトリクスを抽出
- **サブスクリプションフィルター**: ログをリアルタイムで他サービスに配信
- **保持期間（Retention）**: ログを保存する期間
- **Logs Insights**: SQLライクなクエリでログを分析

## 試験頻出ポイント

- [ ] メトリクスフィルターの設定とパターン構文
- [ ] サブスクリプションフィルターの配信先と設定
- [ ] Logs Insightsのクエリ構文
- [ ] ログの保持期間設定（1日〜無期限）
- [ ] クロスアカウントログ集約
- [ ] S3へのログエクスポート
- [ ] ログの暗号化（KMS）
- [ ] Lambda関数のログ設定

## DOP-C02での出題パターン

### パターン1: ログからのメトリクス生成
- **シナリオ**: アプリケーションエラーの発生回数を監視したい
- **ポイント**: メトリクスフィルター + CloudWatchアラーム

### パターン2: リアルタイムログ処理
- **シナリオ**: 特定パターンのログをリアルタイムで処理
- **ポイント**: サブスクリプションフィルター → Lambda/Kinesis

### パターン3: 集中ログ管理
- **シナリオ**: 複数アカウントのログを一箇所に集約
- **ポイント**: クロスアカウントサブスクリプション、宛先ポリシー

### パターン4: ログ分析
- **シナリオ**: 過去のログから特定のエラーパターンを調査
- **ポイント**: Logs Insightsクエリ

### パターン5: コンプライアンス対応
- **シナリオ**: ログを長期間保持してコンプライアンス要件を満たす
- **ポイント**: S3エクスポート + Glacier移行

## ベストプラクティス

1. **保持期間の適切な設定**: コストとコンプライアンス要件のバランス
2. **メトリクスフィルターの活用**: 重要なログパターンを監視
3. **構造化ログの使用**: JSON形式でクエリを容易に
4. **ログの暗号化**: 機密データを含む場合はKMSで暗号化
5. **サブスクリプションフィルターの活用**: リアルタイム処理や集約
6. **コスト最適化**: 不要なログは短い保持期間または削除

## メトリクスフィルター

### パターン構文

```
# 特定の文字列を含む
"ERROR"

# 複数条件（AND）
"ERROR" "database"

# 複数条件（OR）
?ERROR ?WARN

# JSONログ（フィールド指定）
{ $.level = "ERROR" }

# 数値比較
{ $.latency > 1000 }

# 複合条件
{ $.level = "ERROR" && $.statusCode = 500 }
```

### 設定例

```yaml
# CloudFormation
MetricFilter:
  Type: AWS::Logs::MetricFilter
  Properties:
    LogGroupName: /aws/lambda/my-function
    FilterPattern: "ERROR"
    MetricTransformations:
      - MetricName: ErrorCount
        MetricNamespace: MyApp
        MetricValue: "1"
```

## サブスクリプションフィルター

### 配信先

| 配信先 | ユースケース |
|--------|--------------|
| Lambda | リアルタイム処理、アラート |
| Kinesis Data Streams | 大量ログのストリーミング |
| Kinesis Data Firehose | S3/Redshift/Elasticsearch配信 |
| CloudWatch Logs（クロスアカウント） | 集中ログ管理 |

### 制限

- 1ロググループにつき最大2つのサブスクリプションフィルター
- クロスアカウント配信には宛先ポリシーが必要

### クロスアカウント設定

```
# 宛先アカウントで宛先を作成
aws logs put-destination \
  --destination-name CentralLogsDestination \
  --target-arn arn:aws:kinesis:region:DEST_ACCOUNT:stream/logs-stream \
  --role-arn arn:aws:iam::DEST_ACCOUNT:role/LogsDestinationRole

# 宛先ポリシーで送信元アカウントを許可
aws logs put-destination-policy \
  --destination-name CentralLogsDestination \
  --access-policy '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"AWS": "SOURCE_ACCOUNT"},
      "Action": "logs:PutSubscriptionFilter",
      "Resource": "arn:aws:logs:region:DEST_ACCOUNT:destination:CentralLogsDestination"
    }]
  }'
```

## Logs Insights

### 基本クエリ構文

```sql
# フィールドの選択
fields @timestamp, @message

# フィルタリング
filter @message like /ERROR/
filter level = "ERROR"

# ソート
sort @timestamp desc

# 集計
stats count(*) by bin(5m)
stats avg(latency) by service

# 上位N件
limit 100

# パターン抽出
parse @message "user=* action=*" as user, action
```

### クエリ例

```sql
# エラーログの検索
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100

# 5分間隔のエラー数
filter @message like /ERROR/
| stats count(*) as errorCount by bin(5m)

# レイテンシの統計
filter @type = "REPORT"
| stats avg(@duration), max(@duration), min(@duration) by bin(1h)
```

## S3エクスポート

### 方法

1. **手動エクスポート**: `CreateExportTask` API
2. **定期エクスポート**: Lambda + EventBridgeでスケジュール実行

### 注意事項

- エクスポートタスクは非同期処理
- S3バケットにはCloudWatch Logsからの書き込み許可が必要
- 暗号化されたロググループは同じKMSキーでS3も暗号化する必要あり

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| CloudWatch Metrics | メトリクスフィルターでメトリクス生成 |
| Lambda | ログ自動送信、サブスクリプション処理 |
| Kinesis | リアルタイムストリーミング |
| S3 | 長期保存、分析 |
| Athena | S3エクスポート後のクエリ |
| OpenSearch | ログ分析・可視化 |
| EC2 | CloudWatch Agentでログ収集 |
| ECS/Fargate | awslogsドライバーでログ送信 |
| API Gateway | アクセスログ |
| VPC | VPC Flow Logs |

## ログの保持期間オプション

| 期間 | ユースケース |
|------|--------------|
| 1日 | 開発/テスト環境 |
| 1週間〜1ヶ月 | 一般的なトラブルシューティング |
| 3ヶ月〜1年 | コンプライアンス要件 |
| 無期限 | 監査要件（S3エクスポート推奨） |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [CloudWatch Logs ユーザーガイド](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)
- [フィルターパターン構文](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html)
- [Logs Insights クエリ構文](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
