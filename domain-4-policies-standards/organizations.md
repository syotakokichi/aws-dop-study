# AWS Organizations

## 概要

AWS Organizationsは、複数のAWSアカウントを一元管理するためのサービス。アカウントのグループ化、ポリシーの一括適用、一括請求、リソース共有などを通じて、大規模な組織のガバナンスとコスト管理を実現する。

## キーコンセプト

- **組織（Organization）**: AWSアカウントの集合を管理するエンティティ
- **管理アカウント（Management Account）**: 組織の所有者アカウント（旧:マスターアカウント）
- **メンバーアカウント**: 組織に属する個々のAWSアカウント
- **組織単位（OU: Organizational Unit）**: アカウントをグループ化する階層構造
- **ルート**: 組織の最上位コンテナ
- **サービスコントロールポリシー（SCP）**: アカウント/OUに対する権限の境界
- **一括請求（Consolidated Billing）**: 組織全体の請求を統合

## 試験頻出ポイント

- [ ] SCPの動作原理（許可のフィルタリング、DenyListモデル vs AllowListモデル）
- [ ] SCPとIAMポリシーの関係（交差部分のみ有効）
- [ ] OU階層構造とSCPの継承
- [ ] 管理アカウントにはSCPが適用されない
- [ ] 組織証跡、組織Config Rules、組織CloudFormation StackSets
- [ ] AWS RAM（Resource Access Manager）によるリソース共有
- [ ] Control Towerとの連携
- [ ] アカウントの作成・招待・削除

## DOP-C02での出題パターン

### パターン1: 権限の制限
- **シナリオ**: 開発アカウントで特定リージョン以外の利用を禁止
- **ポイント**: SCPでリージョン制限

### パターン2: セキュリティガードレール
- **シナリオ**: 全アカウントでCloudTrailの無効化を防止
- **ポイント**: SCPでcloudtrail:StopLoggingをDeny

### パターン3: マルチアカウント監査
- **シナリオ**: 全アカウントのCloudTrailログを集約
- **ポイント**: 組織証跡

### パターン4: 一括コスト管理
- **シナリオ**: 全アカウントの利用料金を一元管理
- **ポイント**: 一括請求、コスト配分タグ

### パターン5: リソース共有
- **シナリオ**: Transit Gatewayを組織内で共有
- **ポイント**: AWS RAM + Organizations連携

## ベストプラクティス

1. **OU構造の設計**: 用途・環境別に階層化（Production, Development, Sandbox等）
2. **SCPの段階的適用**: 小さく始めて徐々に厳格化
3. **管理アカウントの保護**: 最小限のリソースのみ、ワークロードは配置しない
4. **ログアカウントの分離**: CloudTrail、Config等のログを専用アカウントに集約
5. **セキュリティアカウントの分離**: GuardDuty、Security Hubの管理を専用アカウントで
6. **Control Towerの活用**: ベストプラクティスのガードレールを自動設定

## SCP（サービスコントロールポリシー）

### 動作原理

```
最終権限 = IAMポリシー ∩ SCP ∩ リソースベースポリシー
```

- SCPは**許可を与えない**、許可の**境界を設定**する
- IAMで許可されていても、SCPで許可されていなければ拒否

### SCPモデル

| モデル | デフォルトポリシー | 追加ポリシー |
|--------|-------------------|--------------|
| DenyList（推奨） | FullAWSAccess | 特定アクションをDeny |
| AllowList | FullAWSAccessを削除 | 許可するアクションのみ |

### SCP例：リージョン制限

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyOutsideRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "ap-northeast-1",
            "us-east-1"
          ]
        },
        "ForAllValues:StringNotLike": {
          "aws:PrincipalArn": [
            "arn:aws:iam::*:role/OrganizationAccountAccessRole"
          ]
        }
      }
    }
  ]
}
```

### SCP例：特定アクションの禁止

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCloudTrailDisable",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyLeaveOrganization",
      "Effect": "Deny",
      "Action": "organizations:LeaveOrganization",
      "Resource": "*"
    }
  ]
}
```

## OU設計パターン

### 推奨構造

```
Root
├── Security OU
│   ├── Log Archive Account
│   └── Security Tooling Account
├── Infrastructure OU
│   ├── Shared Services Account
│   └── Network Account
├── Workloads OU
│   ├── Production OU
│   │   ├── App A Prod Account
│   │   └── App B Prod Account
│   └── Non-Production OU
│       ├── Development OU
│       └── Staging OU
└── Sandbox OU
    └── Developer Sandbox Accounts
```

## アカウント操作

### アカウント作成

```bash
# AWS CLIでアカウント作成
aws organizations create-account \
  --email new-account@example.com \
  --account-name "New Account" \
  --iam-user-access-to-billing ALLOW
```

### アカウント移動

```bash
# アカウントを別のOUに移動
aws organizations move-account \
  --account-id 123456789012 \
  --source-parent-id ou-xxxx-yyyyyyyy \
  --destination-parent-id ou-xxxx-zzzzzzzz
```

## AWS RAM連携

### 共有可能なリソース例

| サービス | リソース |
|----------|----------|
| VPC | サブネット、Transit Gateway |
| Route 53 | リゾルバールール |
| License Manager | ライセンス設定 |
| Resource Groups | タグエディター |
| Systems Manager | パラメータストア |

### Organizations連携

```bash
# Organizations内での共有を有効化
aws ram enable-sharing-with-aws-organization
```

## Organizations対応サービス

| サービス | 機能 |
|----------|------|
| CloudTrail | 組織証跡 |
| Config | 組織ルール、コンフォーマンスパック |
| CloudFormation | StackSets（サービスマネージド） |
| GuardDuty | 組織全体の脅威検出 |
| Security Hub | 組織全体のセキュリティ |
| Backup | 組織バックアップポリシー |
| IAM Access Analyzer | 組織全体の分析 |
| Service Catalog | 組織共有ポートフォリオ |

## Control Tower

### 機能

- Organizations上に構築されるアカウント管理フレームワーク
- ベストプラクティスのガードレール（SCP + Config Rules）を自動設定
- Account Factoryでアカウント作成を標準化
- ダッシュボードでコンプライアンスを可視化

### ガードレールタイプ

| タイプ | 実装 | 動作 |
|--------|------|------|
| 予防的（Preventive） | SCP | 操作をブロック |
| 検出的（Detective） | Config Rules | 違反を検出・報告 |

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| IAM | SCP による権限境界 |
| Control Tower | ガードレール、Account Factory |
| CloudTrail | 組織証跡 |
| Config | 組織ルール |
| CloudFormation | サービスマネージド StackSets |
| RAM | 組織内リソース共有 |
| Service Catalog | ポートフォリオ共有 |
| Billing | 一括請求 |
| Cost Explorer | 組織全体のコスト分析 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS Organizations ユーザーガイド](https://docs.aws.amazon.com/organizations/latest/userguide/)
- [SCP 構文と例](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html)
- [AWS Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/)
