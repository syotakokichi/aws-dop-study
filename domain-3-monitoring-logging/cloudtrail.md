# AWS CloudTrail

## 概要

AWS CloudTrailは、AWSアカウントのガバナンス、コンプライアンス、運用監査、リスク監査を実現するサービス。AWSアカウント内で行われたAPI呼び出しやユーザーアクティビティを記録し、セキュリティ分析、リソース変更の追跡、トラブルシューティングに活用できる。

## キーコンセプト

- **証跡（Trail）**: CloudTrailイベントを記録・配信する設定
- **管理イベント（Management Events）**: AWSリソースの管理操作（ControlPlane操作）
- **データイベント（Data Events）**: リソース内のデータ操作（DataPlane操作）
- **インサイトイベント（Insights Events）**: 異常なAPIアクティビティを検出
- **イベント履歴**: 過去90日間の管理イベントを無料で閲覧
- **マルチリージョン証跡**: すべてのリージョンのイベントを収集
- **組織証跡**: AWS Organizations全体のイベントを収集

## 試験頻出ポイント

- [ ] 管理イベントとデータイベントの違い
- [ ] マルチリージョン証跡と単一リージョン証跡
- [ ] 組織証跡（Organization Trail）
- [ ] CloudTrail Lake（長期保存・クエリ）
- [ ] S3バケットへのログ配信と暗号化
- [ ] CloudWatch Logsとの統合
- [ ] イベントセレクター（Advanced Event Selectors）
- [ ] CloudTrailログの整合性検証

## DOP-C02での出題パターン

### パターン1: セキュリティ監査
- **シナリオ**: 誰がいつセキュリティグループを変更したか追跡したい
- **ポイント**: CloudTrailで管理イベントを検索

### パターン2: コンプライアンス要件
- **シナリオ**: 7年間のAPI監査ログを保持する必要がある
- **ポイント**: S3への配信 + Glacier移行、またはCloudTrail Lake

### パターン3: リアルタイムアラート
- **シナリオ**: 特定のAPIが呼び出されたら即座に通知
- **ポイント**: CloudWatch Logsへの配信 + メトリクスフィルター + アラーム

### パターン4: 組織全体の監査
- **シナリオ**: AWS Organizations内の全アカウントを一元監査
- **ポイント**: 組織証跡

### パターン5: データアクセス監査
- **シナリオ**: 機密S3バケットへのアクセスを記録
- **ポイント**: データイベントの有効化（S3オブジェクトレベル）

## ベストプラクティス

1. **マルチリージョン証跡の有効化**: 全リージョンのイベントを収集
2. **ログファイルの整合性検証**: 改ざん検出のため有効化
3. **S3バケットの保護**: バケットポリシー、MFA Delete
4. **暗号化の有効化**: SSE-S3またはSSE-KMS
5. **CloudWatch Logsへの配信**: リアルタイム監視とアラート
6. **組織証跡の使用**: マルチアカウント環境での一元管理

## イベントタイプ

### 管理イベント（Management Events）

| 種類 | 例 |
|------|-----|
| 読み取り | DescribeInstances, ListBuckets |
| 書き込み | RunInstances, CreateBucket, DeleteSecurityGroup |

### データイベント（Data Events）

| サービス | 操作例 |
|----------|--------|
| S3 | GetObject, PutObject, DeleteObject |
| Lambda | Invoke |
| DynamoDB | GetItem, PutItem |
| EBS | CreateSnapshot (Direct APIs) |

### Insightsイベント

- 異常なAPI呼び出しパターンを検出
- 通常のベースラインからの逸脱を特定
- 書き込み管理イベントのみサポート

## 証跡の設定

### マルチリージョン証跡

```yaml
# CloudFormation
MyTrail:
  Type: AWS::CloudTrail::Trail
  Properties:
    TrailName: my-multi-region-trail
    S3BucketName: my-cloudtrail-bucket
    IsMultiRegionTrail: true
    IncludeGlobalServiceEvents: true
    EnableLogFileValidation: true
    KMSKeyId: !Ref MyKmsKey
    CloudWatchLogsLogGroupArn: !GetAtt LogGroup.Arn
    CloudWatchLogsRoleArn: !GetAtt CloudTrailRole.Arn
```

### イベントセレクター（データイベント用）

```yaml
EventSelectors:
  - ReadWriteType: All
    IncludeManagementEvents: true
    DataResources:
      - Type: AWS::S3::Object
        Values:
          - arn:aws:s3:::my-bucket/
      - Type: AWS::Lambda::Function
        Values:
          - arn:aws:lambda
```

### Advanced Event Selectors

```yaml
AdvancedEventSelectors:
  - Name: "Log all S3 GetObject events"
    FieldSelectors:
      - Field: eventCategory
        Equals: ["Data"]
      - Field: resources.type
        Equals: ["AWS::S3::Object"]
      - Field: eventName
        Equals: ["GetObject"]
```

## CloudTrail Lake

### 特徴

- SQLベースのクエリ
- 長期保存（最大7年）
- 複数ソースからのイベント集約
- CloudTrailイベント以外のカスタムイベントも取り込み可能

### クエリ例

```sql
SELECT eventTime, userIdentity.arn, eventName, sourceIPAddress
FROM my-event-data-store
WHERE eventName = 'DeleteBucket'
  AND eventTime > '2024-01-01'
ORDER BY eventTime DESC
```

## ログファイル整合性検証

### 仕組み

1. 毎時間ダイジェストファイルを生成
2. ダイジェストはSHA-256ハッシュを含む
3. RSA署名で改ざんを検出

### 検証コマンド

```bash
aws cloudtrail validate-logs \
  --trail-arn arn:aws:cloudtrail:region:account:trail/my-trail \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| S3 | ログファイルの保存 |
| CloudWatch Logs | リアルタイム配信、メトリクスフィルター |
| EventBridge | イベント駆動でのアクション |
| Athena | S3に保存されたログのクエリ |
| Security Hub | セキュリティ検出結果の統合 |
| GuardDuty | 脅威検出の情報源 |
| Config | 構成変更の追跡 |
| KMS | ログファイルの暗号化 |
| Organizations | 組織証跡 |
| IAM | アクセス制御、アクティビティ追跡 |

## S3バケットポリシー例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::my-bucket"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-bucket/AWSLogs/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}
```

## CloudWatch Logsとの統合

### メトリクスフィルター例（コンソールログイン失敗）

```
{ ($.eventName = ConsoleLogin) && ($.errorMessage = "Failed authentication") }
```

### メトリクスフィルター例（ルートアカウント使用）

```
{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }
```

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS CloudTrail ユーザーガイド](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)
- [CloudTrail Lake](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake.html)
- [ログファイル整合性検証](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html)
