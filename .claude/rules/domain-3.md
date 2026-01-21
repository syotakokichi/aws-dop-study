---
globs:
  - "domain-3-monitoring-logging/**"
  - "**/cloudwatch*"
  - "**/x-ray*"
  - "**/cloudtrail*"
---

# ドメイン3: 監視・ロギング ルール

## 配点: 15%

このドメインの質問に回答する際は、以下のファイルを優先的に参照:

- domain-3-monitoring-logging/cloudwatch.md
- domain-3-monitoring-logging/cloudwatch-logs.md
- domain-3-monitoring-logging/x-ray.md
- domain-3-monitoring-logging/cloudtrail.md

## 頻出トピック

1. **CloudWatch**
   - メトリクスの解像度と保持期間
   - 複合アラーム
   - カスタムメトリクス
   - CloudWatch Agent

2. **CloudWatch Logs**
   - メトリクスフィルター
   - サブスクリプションフィルター
   - Logs Insights クエリ
   - クロスアカウントログ集約

3. **X-Ray**
   - トレース、セグメント、サブセグメント
   - サンプリングルール
   - アノテーション vs メタデータ

4. **CloudTrail**
   - 管理イベント vs データイベント
   - 組織証跡
   - ログファイル整合性検証

## 回答時の注意

- メトリクスの数値（保持期間、解像度）は正確に
- ログ分析は Logs Insights のクエリ例を提示
- CloudTrail と Config の違いを明確に説明
