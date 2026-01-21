# ドメイン2: 構成管理・IaC (17%)

## 概要

Infrastructure as Code (IaC) と構成管理に関するドメイン。
CloudFormation、CDK、Systems Managerが中心。

## 主要サービス

| サービス | 役割 |
|----------|------|
| CloudFormation | IaC（宣言的） |
| CDK | IaC（プログラマティック） |
| Systems Manager | 構成管理・オペレーション |
| OpsWorks | Chef/Puppet統合 |
| Elastic Beanstalk | PaaS |
| SAM | サーバーレスIaC |

## 試験頻出トピック

### CloudFormation
- [ ] スタック操作（作成、更新、削除）
- [ ] ネストスタック
- [ ] クロススタック参照（Export/Import）
- [ ] 変更セット（Change Set）
- [ ] ドリフト検出
- [ ] スタックポリシー
- [ ] カスタムリソース（Lambda）
- [ ] StackSets（マルチアカウント・リージョン）

### CDK
- [ ] Constructs（L1, L2, L3）
- [ ] cdk synth / cdk deploy
- [ ] Aspects
- [ ] Context

### Systems Manager
- [ ] Parameter Store（標準/アドバンスト、SecureString）
- [ ] Automation（ランブック）
- [ ] State Manager
- [ ] Patch Manager
- [ ] Session Manager
- [ ] Inventory

## 学習ファイル

- [cloudformation.md](./cloudformation.md)
- [cdk.md](./cdk.md)
- [systems-manager.md](./systems-manager.md)
- [opsworks.md](./opsworks.md)

## ハンズオン

- [hands-on/](./hands-on/) - 実践記録
