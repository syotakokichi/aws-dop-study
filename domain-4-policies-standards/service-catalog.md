# AWS Service Catalog

## 概要

AWS Service Catalogは、組織内でAWS上のサービスを標準化・管理するためのサービス。IT管理者が承認済みの製品（CloudFormationテンプレート）を作成・管理し、エンドユーザーがセルフサービスでプロビジョニングできる環境を提供する。ガバナンスとアジリティのバランスを実現。

## キーコンセプト

- **ポートフォリオ（Portfolio）**: 製品とその設定を含むコンテナ
- **製品（Product）**: プロビジョニング可能なCloudFormationテンプレート
- **プロビジョニング成果物（Provisioned Product）**: 製品からデプロイされたリソース
- **制約（Constraint）**: 製品の使用方法を制限するルール
- **起動制約（Launch Constraint）**: プロビジョニング時に使用するIAMロール
- **テンプレート制約**: パラメータの選択肢を制限
- **TagOption**: リソースに自動適用するタグ

## 試験頻出ポイント

- [ ] ポートフォリオと製品の関係
- [ ] 起動制約（Launch Constraint）の役割と設定
- [ ] テンプレート制約によるパラメータ制限
- [ ] Organizations連携による組織全体への共有
- [ ] アクセス制御（IAMプリンシパル、グループへの付与）
- [ ] 製品のバージョン管理
- [ ] プロビジョニング成果物の更新と終了
- [ ] StackSets製品（マルチアカウント/リージョン）

## DOP-C02での出題パターン

### パターン1: セルフサービスプロビジョニング
- **シナリオ**: 開発者に承認済みリソースのみを作成させたい
- **ポイント**: Service Catalog製品 + 起動制約

### パターン2: ガバナンスとコンプライアンス
- **シナリオ**: すべてのEC2インスタンスに特定のタグを強制
- **ポイント**: TagOption + テンプレート制約

### パターン3: 組織全体での標準化
- **シナリオ**: 全アカウントで同じ製品カタログを提供
- **ポイント**: Organizations連携、ポートフォリオ共有

### パターン4: 権限の最小化
- **シナリオ**: 開発者にCloudFormation直接アクセスを与えずにリソース作成を許可
- **ポイント**: 起動制約 + 製品アクセス権限のみ付与

## ベストプラクティス

1. **製品のバージョン管理**: 変更履歴を追跡し、ロールバック可能に
2. **起動制約の活用**: ユーザーに過剰な権限を与えない
3. **テンプレート制約の設定**: 選択可能なパラメータを制限
4. **TagOptionの活用**: コスト配分タグを自動適用
5. **Organizations連携**: 組織全体でポートフォリオを共有
6. **承認プロセスの確立**: 製品追加時のレビューフロー

## 起動制約（Launch Constraint）

### 目的

- ユーザーは製品を起動する権限のみ必要
- 実際のリソース作成は指定したIAMロールで実行
- ユーザーにCloudFormation等の直接権限を与えない

### 設定

```yaml
# 起動制約で使用するIAMロール
LaunchRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: servicecatalog.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/AmazonEC2FullAccess
      - arn:aws:iam::aws:policy/AWSCloudFormationFullAccess
```

### 権限フロー

```
ユーザー → Service Catalog (Launch Product権限のみ)
                ↓
        起動制約のIAMロールでCloudFormation実行
                ↓
        リソースが作成される
```

## テンプレート制約

### 用途

- CloudFormationパラメータの選択肢を制限
- ユーザーが選べる値を事前に定義

### 例: インスタンスタイプの制限

```json
{
  "Rules": {
    "InstanceType": {
      "Assertions": [
        {
          "Assert": {
            "Fn::Contains": [
              ["t3.micro", "t3.small", "t3.medium"],
              {"Ref": "InstanceType"}
            ]
          },
          "AssertDescription": "インスタンスタイプはt3.micro, t3.small, t3.mediumのみ"
        }
      ]
    }
  }
}
```

## TagOption

### 用途

- ポートフォリオ/製品にタグを関連付け
- プロビジョニング時に自動適用
- コスト配分、リソース識別に活用

### 設定例

```yaml
TagOption:
  Type: AWS::ServiceCatalog::TagOption
  Properties:
    Key: CostCenter
    Value: "CC-12345"

TagOptionAssociation:
  Type: AWS::ServiceCatalog::TagOptionAssociation
  Properties:
    ResourceId: !Ref MyPortfolio
    TagOptionId: !Ref TagOption
```

## Organizations連携

### 共有方法

1. **ポートフォリオを組織/OUと共有**
2. 受信アカウントでポートフォリオをインポート
3. ローカルユーザーにアクセス権限を付与

### 設定

```yaml
PortfolioShare:
  Type: AWS::ServiceCatalog::PortfolioPrincipalAssociation
  Properties:
    PortfolioId: !Ref MyPortfolio
    PrincipalARN: arn:aws:organizations::123456789012:organization/o-xxxxxxxxxx
    PrincipalType: IAM
```

## 製品タイプ

| タイプ | 説明 |
|--------|------|
| CloudFormation | 標準的なCFnテンプレート |
| CloudFormation StackSets | マルチアカウント/リージョンデプロイ |
| Terraform | Terraform Cloud連携 |
| External | 外部ツール連携 |

## アクセス制御

### 必要な権限

| アクション | 説明 |
|------------|------|
| servicecatalog:ListPortfolios | ポートフォリオ一覧 |
| servicecatalog:ListProducts | 製品一覧 |
| servicecatalog:ProvisionProduct | 製品のプロビジョニング |
| servicecatalog:DescribeRecord | プロビジョニング状況確認 |
| servicecatalog:TerminateProvisionedProduct | リソースの削除 |
| servicecatalog:UpdateProvisionedProduct | リソースの更新 |

### IAMポリシー例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "servicecatalog:ListPortfolios",
        "servicecatalog:SearchProducts",
        "servicecatalog:DescribeProduct",
        "servicecatalog:ProvisionProduct",
        "servicecatalog:TerminateProvisionedProduct"
      ],
      "Resource": "*"
    }
  ]
}
```

## プロビジョニングワークフロー

```
1. ユーザーがポートフォリオから製品を選択
2. パラメータを入力（テンプレート制約の範囲内）
3. Service Catalogが起動制約のロールでCloudFormation実行
4. リソースがプロビジョニング
5. ユーザーはプロビジョニング成果物として管理
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| CloudFormation | 製品テンプレート |
| IAM | 起動制約、アクセス制御 |
| Organizations | ポートフォリオ共有 |
| Config | プロビジョニングリソースの監視 |
| CloudTrail | 操作の監査ログ |
| Terraform Cloud | Terraform製品タイプ |

## ユースケース

| シナリオ | 解決策 |
|----------|--------|
| 開発者にDB作成を許可するがスペック制限 | テンプレート制約 + 起動制約 |
| 全リソースにコストセンタータグを強制 | TagOption |
| 組織全体で標準VPCテンプレートを提供 | Organizations共有 |
| 複数リージョンに同時デプロイ | StackSets製品タイプ |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS Service Catalog ユーザーガイド](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/)
- [製品の作成](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/productmgmt-cloudresource.html)
- [制約の管理](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/constraints.html)
