# ドメイン3: 監視・ロギング (15%)

## 概要

システムの可観測性（Observability）に関するドメイン。
メトリクス、ログ、トレースの収集・分析が中心。

## 主要サービス

| サービス | 役割 |
|----------|------|
| CloudWatch Metrics | メトリクス収集・可視化 |
| CloudWatch Logs | ログ集約・分析 |
| CloudWatch Alarms | アラート |
| X-Ray | 分散トレーシング |
| CloudTrail | API監査ログ |
| VPC Flow Logs | ネットワークトラフィック |

## 試験頻出トピック

### CloudWatch Metrics
- [ ] 標準メトリクス vs カスタムメトリクス
- [ ] 高解像度メトリクス
- [ ] メトリクス数学（Metric Math）
- [ ] ダッシュボード
- [ ] Contributor Insights

### CloudWatch Logs
- [ ] ロググループ・ログストリーム
- [ ] メトリクスフィルター
- [ ] サブスクリプションフィルター（Kinesis, Lambda, OpenSearch）
- [ ] Logs Insights（クエリ言語）
- [ ] ログ保持期間

### CloudWatch Alarms
- [ ] 複合アラーム（Composite Alarm）
- [ ] アラームアクション（SNS, Auto Scaling, EC2）
- [ ] アノマリー検出

### X-Ray
- [ ] トレース・セグメント・サブセグメント
- [ ] サービスマップ
- [ ] サンプリングルール
- [ ] X-Ray daemon

### CloudTrail
- [ ] 管理イベント vs データイベント
- [ ] マルチリージョン・組織証跡
- [ ] CloudWatch Logs統合
- [ ] Insights

## 学習ファイル

- [cloudwatch.md](./cloudwatch.md)
- [cloudwatch-logs.md](./cloudwatch-logs.md)
- [x-ray.md](./x-ray.md)
- [cloudtrail.md](./cloudtrail.md)

## ハンズオン

- [hands-on/](./hands-on/) - 実践記録
