# ドメイン4: ポリシー・標準 (10%)

## 概要

セキュリティ、コンプライアンス、ガバナンスに関するドメイン。
AWS Organizationsとガードレールの設計が中心。

## 主要サービス

| サービス | 役割 |
|----------|------|
| AWS Config | 構成変更追跡・コンプライアンス |
| Organizations | マルチアカウント管理 |
| Control Tower | ランディングゾーン |
| Service Catalog | 標準化されたサービス提供 |
| License Manager | ライセンス管理 |
| Trusted Advisor | ベストプラクティスチェック |

## 試験頻出トピック

### AWS Config
- [ ] Configルール（マネージド/カスタム）
- [ ] 修復アクション（自動/手動）
- [ ] コンフォーマンスパック
- [ ] アグリゲーター
- [ ] 構成履歴・スナップショット

### Organizations
- [ ] サービスコントロールポリシー（SCP）
- [ ] 組織単位（OU）
- [ ] AWS サービスとの統合
- [ ] タグポリシー
- [ ] バックアップポリシー

### Control Tower
- [ ] ガードレール（予防的/検出的）
- [ ] Account Factory
- [ ] ランディングゾーン

### Service Catalog
- [ ] ポートフォリオ
- [ ] プロダクト
- [ ] 制約（起動/テンプレート/通知）
- [ ] TagOptions

## 学習ファイル

- [aws-config.md](./aws-config.md)
- [organizations.md](./organizations.md)
- [service-catalog.md](./service-catalog.md)

## ハンズオン

- [hands-on/](./hands-on/) - 実践記録
