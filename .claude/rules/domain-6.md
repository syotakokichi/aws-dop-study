---
globs:
  - "domain-6-ha-fault-tolerance/**"
  - "**/auto-scaling*"
  - "**/elb*"
  - "**/route53*"
  - "**/disaster-recovery*"
---

# ドメイン6: 高可用性・耐障害性 ルール

## 配点: 18%

このドメインの質問に回答する際は、以下のファイルを優先的に参照:

- domain-6-ha-fault-tolerance/auto-scaling.md
- domain-6-ha-fault-tolerance/elb.md
- domain-6-ha-fault-tolerance/route53.md
- domain-6-ha-fault-tolerance/disaster-recovery.md

## 頻出トピック

1. **Auto Scaling**
   - スケーリングポリシー（ターゲット追跡、ステップ、予測）
   - ウォームプール
   - インスタンスリフレッシュ
   - ライフサイクルフック

2. **ELB**
   - ALB vs NLB vs GLB の選択
   - ヘルスチェック設定
   - クロスゾーン負荷分散
   - SSL/TLS 終端

3. **Route 53**
   - ルーティングポリシー
   - フェイルオーバー設定
   - ヘルスチェック
   - Resolver（ハイブリッドDNS）

4. **DR戦略**
   - Backup & Restore / Pilot Light / Warm Standby / Active-Active
   - RTO / RPO の違い
   - クロスリージョンレプリケーション

## 回答時の注意

- DR戦略は RTO/RPO とコストのトレードオフで説明
- ルーティングポリシーの選択理由を明確に
- Multi-AZ と Multi-Region の違いを区別
