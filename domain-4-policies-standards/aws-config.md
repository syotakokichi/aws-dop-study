# AWS Config

## 概要

AWS Configは、AWSリソースの構成を記録、評価、監査するサービス。リソースの構成変更を継続的に追跡し、設定したルールに対するコンプライアンス状態を評価する。構成履歴の保存、コンプライアンスの可視化、自動修復が可能。

## キーコンセプト

- **構成アイテム（Configuration Item）**: リソースの特定時点での構成スナップショット
- **構成レコーダー**: リソースの構成変更を記録するコンポーネント
- **配信チャネル**: 構成スナップショットと履歴をS3に配信
- **Configルール**: リソース構成のコンプライアンスを評価
- **コンフォーマンスパック**: 複数のルールをパッケージ化
- **アグリゲーター**: マルチアカウント/リージョンのデータを集約
- **修復アクション（Remediation）**: 非準拠リソースを自動修正

## 試験頻出ポイント

- [ ] マネージドルール vs カスタムルール
- [ ] 修復アクションの設定（手動/自動）
- [ ] コンフォーマンスパックによる一括デプロイ
- [ ] アグリゲーターによるマルチアカウント集約
- [ ] 高度なクエリ（SQL）
- [ ] 構成履歴と構成スナップショット
- [ ] CloudTrailとの違い（構成 vs API呼び出し）
- [ ] 組織全体への展開（Organization Config Rules）

## DOP-C02での出題パターン

### パターン1: コンプライアンス監視
- **シナリオ**: 全S3バケットが暗号化されているか確認したい
- **ポイント**: マネージドルール（s3-bucket-server-side-encryption-enabled）

### パターン2: 自動修復
- **シナリオ**: 非準拠のセキュリティグループを自動で修正
- **ポイント**: Configルール + Systems Manager Automation修復

### パターン3: マルチアカウントコンプライアンス
- **シナリオ**: 組織全体のコンプライアンス状況を一元管理
- **ポイント**: アグリゲーター + Organization Config Rules

### パターン4: 構成変更の追跡
- **シナリオ**: リソースの過去の構成を調査
- **ポイント**: 構成履歴、構成タイムライン

### パターン5: CloudFormationドリフト検出
- **シナリオ**: CloudFormationスタックとの差異を検出
- **ポイント**: cloudformation-stack-drift-detection-check ルール

## ベストプラクティス

1. **全リソースタイプの記録**: すべてのリソースを対象に
2. **マネージドルールの活用**: AWS提供の300以上のルールを活用
3. **自動修復の設定**: 非準拠リソースを自動修正
4. **コンフォーマンスパックの使用**: 業界標準への準拠を効率化
5. **アグリゲーターの設定**: マルチアカウント環境での一元管理
6. **S3への配信と保持**: コンプライアンス要件に応じた保存期間

## Configルール

### マネージドルール例

| ルール名 | 評価内容 |
|----------|----------|
| s3-bucket-server-side-encryption-enabled | S3暗号化 |
| ec2-instance-managed-by-ssm | SSM管理下 |
| rds-instance-public-access-check | RDSパブリックアクセス |
| iam-password-policy | IAMパスワードポリシー |
| root-account-mfa-enabled | ルートMFA |
| cloudtrail-enabled | CloudTrail有効化 |
| vpc-flow-logs-enabled | VPC Flow Logs |
| ebs-optimized-instance | EBS最適化 |
| encrypted-volumes | EBSボリューム暗号化 |
| required-tags | 必須タグ |

### カスタムルール

```python
# Lambda関数でカスタム評価ロジック
import json
import boto3

def lambda_handler(event, context):
    config = boto3.client('config')

    # 評価対象のリソース構成
    configuration_item = json.loads(event['invokingEvent'])['configurationItem']

    # カスタム評価ロジック
    compliance_type = evaluate_compliance(configuration_item)

    # 評価結果を返却
    config.put_evaluations(
        Evaluations=[{
            'ComplianceResourceType': configuration_item['resourceType'],
            'ComplianceResourceId': configuration_item['resourceId'],
            'ComplianceType': compliance_type,  # COMPLIANT, NON_COMPLIANT, NOT_APPLICABLE
            'OrderingTimestamp': configuration_item['configurationItemCaptureTime']
        }],
        ResultToken=event['resultToken']
    )
```

### ルールトリガータイプ

| タイプ | 説明 | ユースケース |
|--------|------|--------------|
| 構成変更 | リソース変更時に評価 | リアルタイム検出 |
| 定期実行 | スケジュールで評価 | 定期的なコンプライアンスチェック |

## 修復アクション

### 設定方法

1. Configルールに修復アクションを関連付け
2. Systems Manager Automationドキュメントを指定
3. 手動または自動実行を選択

### 修復例

| 非準拠状態 | 修復アクション |
|------------|----------------|
| S3バケット暗号化なし | AWS-EnableS3BucketEncryption |
| EBSボリューム暗号化なし | AWS-EncryptEBSVolume（スナップショット経由） |
| セキュリティグループ全開放 | カスタム Automation |
| タグ未設定 | AWS-SetRequiredTags |

### 自動修復の設定

```yaml
# CloudFormation
ConfigRule:
  Type: AWS::Config::ConfigRule
  Properties:
    ConfigRuleName: s3-bucket-encryption
    Source:
      Owner: AWS
      SourceIdentifier: S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED

RemediationConfiguration:
  Type: AWS::Config::RemediationConfiguration
  Properties:
    ConfigRuleName: !Ref ConfigRule
    TargetType: SSM_DOCUMENT
    TargetId: AWS-EnableS3BucketEncryption
    Automatic: true
    MaximumAutomaticAttempts: 5
    RetryAttemptSeconds: 60
    Parameters:
      BucketName:
        ResourceValue:
          Value: RESOURCE_ID
```

## コンフォーマンスパック

### 用途

- 複数のConfigルールをパッケージ化
- 組織全体に一括デプロイ
- 業界標準への準拠（CIS、PCI-DSS等）

### デプロイ方法

```yaml
# コンフォーマンスパックテンプレート
Resources:
  S3BucketEncryptionRule:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: s3-bucket-encryption
      Source:
        Owner: AWS
        SourceIdentifier: S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED

  S3BucketPublicReadRule:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: s3-bucket-public-read
      Source:
        Owner: AWS
        SourceIdentifier: S3_BUCKET_PUBLIC_READ_PROHIBITED
```

## アグリゲーター

### 機能

- マルチアカウント/マルチリージョンのデータを集約
- 組織全体のコンプライアンスビュー
- 高度なクエリで横断的な分析

### 設定

```yaml
ConfigurationAggregator:
  Type: AWS::Config::ConfigurationAggregator
  Properties:
    ConfigurationAggregatorName: org-aggregator
    OrganizationAggregationSource:
      RoleArn: !GetAtt AggregatorRole.Arn
      AllAwsRegions: true
```

## 高度なクエリ

```sql
-- 暗号化されていないS3バケットを検索
SELECT
  resourceId,
  resourceType,
  configuration.serverSideEncryptionConfiguration
WHERE
  resourceType = 'AWS::S3::Bucket'
  AND configuration.serverSideEncryptionConfiguration IS NULL

-- パブリックIPを持つEC2インスタンス
SELECT
  resourceId,
  configuration.publicIpAddress
WHERE
  resourceType = 'AWS::EC2::Instance'
  AND configuration.publicIpAddress IS NOT NULL
```

## Config vs CloudTrail

| 項目 | AWS Config | CloudTrail |
|------|------------|------------|
| 記録対象 | リソースの構成 | API呼び出し |
| 目的 | コンプライアンス評価 | 監査・セキュリティ |
| 質問 | 「リソースはどう設定されているか」 | 「誰が何をしたか」 |
| 評価 | ルールベースの自動評価 | なし |
| 修復 | 自動修復アクション | なし |

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| S3 | 構成履歴・スナップショット保存 |
| SNS | 構成変更通知 |
| Systems Manager | 修復アクション |
| CloudTrail | API呼び出しとの相関 |
| Security Hub | セキュリティ検出結果の統合 |
| Organizations | 組織全体のルールデプロイ |
| Lambda | カスタムルールの評価ロジック |
| EventBridge | 構成変更イベントの処理 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS Config ユーザーガイド](https://docs.aws.amazon.com/config/latest/developerguide/)
- [マネージドルール一覧](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
- [コンフォーマンスパック](https://docs.aws.amazon.com/config/latest/developerguide/conformance-packs.html)
