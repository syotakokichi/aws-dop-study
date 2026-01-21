---
globs:
  - "domain-4-policies-standards/**"
  - "**/aws-config*"
  - "**/organizations*"
  - "**/service-catalog*"
  - "**/scp*"
---

# ドメイン4: ポリシー・標準 ルール

## 配点: 10%（最小配点だが重要）

このドメインの質問に回答する際は、以下のファイルを優先的に参照:

- domain-4-policies-standards/aws-config.md
- domain-4-policies-standards/organizations.md
- domain-4-policies-standards/service-catalog.md

## 頻出トピック

1. **AWS Config**
   - マネージドルール vs カスタムルール
   - 修復アクション（自動修復）
   - コンフォーマンスパック
   - アグリゲーター

2. **Organizations**
   - SCP（サービスコントロールポリシー）
   - OU 設計
   - 管理アカウントの特性（SCPが適用されない）
   - 組織証跡、組織 Config Rules

3. **Service Catalog**
   - ポートフォリオと製品
   - 起動制約（Launch Constraint）
   - テンプレート制約

## 回答時の注意

- SCP は「許可を与える」のではなく「許可の境界を設定」と説明
- Config vs CloudTrail の違いを明確に
- Service Catalog は権限委譲のユースケースで説明
