# AWS CloudFormation

## 概要

AWS CloudFormationは、インフラストラクチャをコード（IaC）として管理するためのサービス。JSONまたはYAML形式のテンプレートでAWSリソースを定義し、スタックとしてプロビジョニング・管理できる。リソースの依存関係を自動的に処理し、一貫性のある環境構築を実現する。

## キーコンセプト

- **テンプレート**: リソースを定義するJSONまたはYAMLファイル
- **スタック**: テンプレートから作成されたリソースの集合
- **変更セット（Change Set）**: スタック更新前に変更内容をプレビュー
- **ドリフト検出**: 実際のリソース状態とテンプレート定義の差異を検出
- **ネストスタック（Nested Stack）**: 他のスタックから参照されるスタック
- **スタックセット（StackSets）**: 複数アカウント/リージョンへの一括デプロイ
- **クロススタック参照**: Export/Importによるスタック間の値共有
- **カスタムリソース**: Lambda関数で独自リソースを管理

## 試験頻出ポイント

- [ ] テンプレート構造（Parameters, Mappings, Conditions, Resources, Outputs）
- [ ] 組み込み関数（!Ref, !GetAtt, !Sub, !Join, !If, !ImportValue）
- [ ] 変更セットの使い方と重要性
- [ ] ドリフト検出の仕組みと対応
- [ ] スタックポリシーによるリソース保護
- [ ] DeletionPolicy（Retain, Snapshot, Delete）
- [ ] UpdateReplacePolicy（Retain, Snapshot, Delete）
- [ ] スタックセットによるマルチアカウント/マルチリージョンデプロイ
- [ ] ネストスタックとクロススタック参照の使い分け
- [ ] カスタムリソースの実装

## DOP-C02での出題パターン

### パターン1: ドリフト検出と修復
- **シナリオ**: 手動変更されたリソースをテンプレート定義に戻したい
- **ポイント**: ドリフト検出 → 手動修正またはスタック更新

### パターン2: マルチアカウントデプロイ
- **シナリオ**: 複数のAWSアカウントに同じインフラを展開
- **ポイント**: StackSets + Organizations連携

### パターン3: リソースの保護
- **シナリオ**: RDSインスタンスが誤って削除されないようにしたい
- **ポイント**: DeletionPolicy: Retain、スタックポリシー

### パターン4: 段階的更新
- **シナリオ**: 更新の影響を確認してから適用したい
- **ポイント**: 変更セットの作成と確認

### パターン5: 外部リソースの管理
- **シナリオ**: CloudFormationでサポートされていないリソースを管理
- **ポイント**: カスタムリソース + Lambda

## ベストプラクティス

1. **変更セットを必ず使用**: 本番環境では更新前に変更内容を確認
2. **DeletionPolicyの設定**: 重要なリソースにはRetainまたはSnapshotを設定
3. **パラメータの活用**: 環境ごとの差異はパラメータで吸収
4. **スタックの分割**: 機能単位でスタックを分割し、ネストスタックで組み合わせ
5. **ドリフト検出の定期実行**: Config Rulesで自動検出
6. **スタックポリシーの適用**: 重要リソースの意図しない更新を防止

## テンプレート構造

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Sample template

Parameters:
  Environment:
    Type: String
    AllowedValues:
      - dev
      - stg
      - prod

Mappings:
  RegionMap:
    ap-northeast-1:
      AMI: ami-12345678

Conditions:
  IsProd: !Equals [!Ref Environment, prod]

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      BucketName: !Sub '${Environment}-my-bucket'

Outputs:
  BucketName:
    Value: !Ref MyBucket
    Export:
      Name: !Sub '${AWS::StackName}-BucketName'
```

## 主要な組み込み関数

| 関数 | 用途 | 例 |
|------|------|-----|
| !Ref | リソースID/パラメータ値を参照 | `!Ref MyBucket` |
| !GetAtt | リソースの属性を取得 | `!GetAtt MyBucket.Arn` |
| !Sub | 文字列内の変数を置換 | `!Sub '${AWS::Region}-bucket'` |
| !Join | 配列を結合 | `!Join ['-', [a, b, c]]` |
| !Split | 文字列を分割 | `!Split [',', 'a,b,c']` |
| !Select | 配列から要素を選択 | `!Select [0, !Ref SubnetIds]` |
| !If | 条件分岐 | `!If [IsProd, t3.large, t3.small]` |
| !ImportValue | 他スタックのエクスポート値を参照 | `!ImportValue VPCId` |
| !FindInMap | Mappingsから値を取得 | `!FindInMap [RegionMap, !Ref 'AWS::Region', AMI]` |

## DeletionPolicy / UpdateReplacePolicy

| ポリシー | 動作 |
|----------|------|
| Delete | リソースを削除（デフォルト） |
| Retain | リソースを残す（スタックから切り離し） |
| Snapshot | スナップショットを作成してから削除（RDS, EBS等） |

## スタックセット

### 用途
- マルチアカウント展開（Organizations連携）
- マルチリージョン展開
- ガードレールの一括適用

### 権限モデル
| モデル | 説明 |
|--------|------|
| Self-managed | 管理者/実行ロールを手動作成 |
| Service-managed | Organizations連携で自動管理 |

## ネストスタック vs クロススタック参照

| 項目 | ネストスタック | クロススタック参照 |
|------|----------------|-------------------|
| 関係性 | 親子関係 | 独立 |
| ライフサイクル | 親と共に更新/削除 | 個別に管理 |
| ユースケース | コンポーネントの再利用 | 共通リソースの共有 |
| 制限 | スタック数の制限に注意 | Export名はリージョン内で一意 |

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| AWS Config | ドリフト検出の自動化 |
| Systems Manager | パラメータストアからの値取得 |
| Secrets Manager | シークレットの参照 |
| Lambda | カスタムリソース |
| CodePipeline | CI/CDでのデプロイ |
| S3 | テンプレートの保存 |
| IAM | スタック操作の権限管理 |
| Organizations | StackSetsでのマルチアカウント管理 |
| Service Catalog | 承認済みテンプレートの提供 |

## カスタムリソース

```yaml
Resources:
  CustomResource:
    Type: Custom::MyResource
    Properties:
      ServiceToken: !GetAtt MyLambdaFunction.Arn
      Parameter1: value1

  MyLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.11
      Handler: index.handler
      Code:
        ZipFile: |
          import cfnresponse
          def handler(event, context):
            # CREATE, UPDATE, DELETE の処理
            cfnresponse.send(event, context, cfnresponse.SUCCESS, {})
```

## スタック更新時の動作

| 更新タイプ | 説明 |
|------------|------|
| Update with No Interruption | サービス中断なし |
| Update with Some Interruption | 一時的な中断あり |
| Replacement | リソースを再作成 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS CloudFormation ユーザーガイド](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/)
- [テンプレートリファレンス](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)
- [組み込み関数リファレンス](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)
- [ベストプラクティス](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
