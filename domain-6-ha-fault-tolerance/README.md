# ドメイン6: 高可用性・耐障害性 (18%)

## 概要

システムの可用性と災害復旧に関するドメイン。
マルチAZ/リージョン設計、Auto Scaling、DR戦略が中心。

## 主要サービス

| サービス | 役割 |
|----------|------|
| Auto Scaling | 自動スケーリング |
| ELB (ALB/NLB/GWLB) | 負荷分散 |
| Route 53 | DNS・ヘルスチェック |
| RDS | マルチAZ・リードレプリカ |
| ElastiCache | キャッシュ・レプリケーション |
| DynamoDB | グローバルテーブル |
| S3 | クロスリージョンレプリケーション |
| Resilience Hub | 回復力評価 |

## 試験頻出トピック

### Auto Scaling
- [ ] 起動テンプレート vs 起動設定
- [ ] スケーリングポリシー（ターゲット追跡/ステップ/シンプル）
- [ ] スケジュールスケーリング
- [ ] 予測スケーリング
- [ ] ウォームプール
- [ ] ライフサイクルフック
- [ ] インスタンス更新（Instance Refresh）

### ELB
- [ ] ALB vs NLB vs GWLB
- [ ] ターゲットグループ
- [ ] ヘルスチェック
- [ ] スティッキーセッション
- [ ] Connection Draining
- [ ] クロスゾーン負荷分散

### Route 53
- [ ] ルーティングポリシー（シンプル/加重/レイテンシ/フェイルオーバー/地理的位置/地理的近接性/マルチバリュー）
- [ ] ヘルスチェック
- [ ] プライベートホストゾーン

### DR戦略（RTO/RPO順）
1. **Backup & Restore**: 最低コスト、最長RTO
2. **Pilot Light**: 最小限のコアインフラを常時稼働
3. **Warm Standby**: 縮小版の本番環境
4. **Active-Active (Multi-Site)**: 最高コスト、最短RTO

### データベースHA
- [ ] RDS マルチAZ（同期レプリケーション）
- [ ] RDS リードレプリカ（非同期）
- [ ] Aurora グローバルデータベース
- [ ] DynamoDB グローバルテーブル
- [ ] ElastiCache レプリケーション

## 学習ファイル

- [auto-scaling.md](./auto-scaling.md)
- [elb.md](./elb.md)
- [route53.md](./route53.md)
- [multi-az-region.md](./multi-az-region.md)
- [disaster-recovery.md](./disaster-recovery.md)

## ハンズオン

- [hands-on/](./hands-on/) - 実践記録
