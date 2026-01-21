---
globs:
  - "domain-2-config-management/**"
  - "**/cloudformation*"
  - "**/cdk*"
  - "**/systems-manager*"
  - "**/ssm*"
  - "**/opsworks*"
---

# ドメイン2: 構成管理・IaC ルール

## 配点: 17%

このドメインの質問に回答する際は、以下のファイルを優先的に参照:

- domain-2-config-management/cloudformation.md
- domain-2-config-management/cdk.md
- domain-2-config-management/systems-manager.md
- domain-2-config-management/opsworks.md

## 頻出トピック

1. **CloudFormation**
   - 変更セットとドリフト検出
   - ネストスタック vs クロススタック参照
   - カスタムリソース
   - DeletionPolicy / UpdateReplacePolicy

2. **Systems Manager**
   - Parameter Store vs Secrets Manager
   - Session Manager（SSHレス接続）
   - Automation ランブック
   - Patch Manager

3. **CDK**
   - Construct レベル（L1, L2, L3）
   - cdk synth / cdk diff / cdk deploy
   - CDK Pipelines

## 回答時の注意

- CloudFormation テンプレートは YAML 形式で例示
- 組み込み関数（!Ref, !GetAtt, !Sub 等）を適切に使用
- OpsWorks Stacks は新規作成終了のため、移行先を提案
