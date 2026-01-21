# SRE概念とAWSサービスの対応表

> SRE本で学んだ概念をAWSサービスに適用する際のマッピング

## 第1部: 導入

### 第1章: イントロダクション

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| SREの役割 | DevOps Engineer, Cloud Operations | AWSでの職種 |
| 50%ルール（運用 vs 開発） | - | 組織設計の考え方 |

### 第2章: Googleの本番環境

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| データセンター | リージョン、AZ | 地理的分散 |
| サーバー管理 | EC2, ECS, EKS, Lambda | コンピュート選択 |
| ネットワーク | VPC, Transit Gateway, Direct Connect | ネットワーク設計 |
| ストレージ | S3, EBS, EFS, FSx | ストレージ選択 |

---

## 第2部: 原則

### 第3章: リスクの受容

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| 可用性目標 | CloudWatch SLO, Synthetics | SLO設定・監視 |
| エラーバジェット | CloudWatch メトリクス計算 | カスタムダッシュボード |
| リスク分析 | Resilience Hub | 回復力評価 |

### 第4章: サービスレベル目標

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| SLI（指標） | CloudWatch Metrics | レイテンシ、可用性等 |
| SLO（目標） | CloudWatch Application Signals | SLO追跡 |
| SLA（契約） | - | ビジネス契約 |

### 第5章: トイルの排除

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| 自動化 | Systems Manager Automation | ランブック |
| 反復作業の削減 | Lambda, Step Functions | イベント駆動自動化 |
| セルフサービス | Service Catalog | 標準化されたプロビジョニング |

### 第6章: 分散システムのモニタリング

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| 4つのゴールデンシグナル | CloudWatch Metrics | |
| - レイテンシ | ALB Latency, X-Ray | 応答時間 |
| - トラフィック | ALB RequestCount | リクエスト数 |
| - エラー | ALB HTTPCode_5XX | エラー率 |
| - サチュレーション | CPU/Memory Utilization | リソース飽和度 |
| 分散トレーシング | X-Ray | トレース・セグメント |
| ダッシュボード | CloudWatch Dashboards | 可視化 |

### 第7章: Googleにおける自動化の進化

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| 自動化の段階 | SSM Automation | 段階的自動化 |
| 構成管理 | CloudFormation, CDK | IaC |
| イミュータブルインフラ | AMI, コンテナイメージ | 変更不可なインフラ |

### 第8章: リリースエンジニアリング

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| CI/CD | CodePipeline, CodeBuild, CodeDeploy | パイプライン |
| カナリアリリース | CodeDeploy (カナリア設定) | 段階的リリース |
| Blue/Greenデプロイ | CodeDeploy, ECS Blue/Green | ゼロダウンタイム |
| ロールバック | CodeDeploy ロールバック | 自動/手動 |

### 第9章: シンプルさ

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| 複雑さの管理 | Well-Architected Review | アーキテクチャ評価 |
| 技術的負債 | - | 設計原則 |

---

## 第3部: プラクティス

### 第10章: 時系列データからの実践的アラート

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| アラート設計 | CloudWatch Alarms | 閾値設定 |
| 複合アラーム | Composite Alarms | 複数条件 |
| アノマリー検出 | CloudWatch Anomaly Detection | ML検出 |
| 通知 | SNS, ChatOps (Slack) | アラート配信 |

### 第11章: オンコール

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| オンコールローテーション | Incident Manager | 対応者管理 |
| エスカレーション | Incident Manager | 段階的エスカレ |
| ページング | SNS, PagerDuty連携 | 通知 |

### 第12章: 効果的なトラブルシューティング

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| ログ分析 | CloudWatch Logs Insights | クエリ分析 |
| トレース分析 | X-Ray | 分散トレーシング |
| メトリクス相関 | CloudWatch Contributor Insights | 相関分析 |

### 第13章: 緊急対応

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| インシデント対応 | Incident Manager | 対応プロセス |
| ランブック | SSM Automation Documents | 手順書 |
| コミュニケーション | SNS, ChatOps | 通知・連携 |

### 第14章: インシデントの管理

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| インシデント管理 | Incident Manager | 統合管理 |
| 役割定義 | Incident Manager Response Plans | 役割設定 |
| タイムライン | Incident Manager Timeline | 時系列記録 |

### 第15章: ポストモーテム文化

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| ポストモーテム | Incident Manager Analysis | 振り返り |
| 根本原因分析 | CloudWatch Logs Insights, X-Ray | 原因特定 |
| アクションアイテム | - | 改善活動 |

### 第16章: 障害の追跡

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| 障害追跡 | CloudTrail, Config | 変更追跡 |
| 影響範囲分析 | X-Ray Service Map | 依存関係可視化 |

### 第17章: 信頼性のためのテスト

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| 負荷テスト | - (外部ツール) | JMeter, Locust等 |
| カオスエンジニアリング | Fault Injection Simulator | 障害注入 |
| 統合テスト | CodeBuild | CI/CD内テスト |

---

## 第4部: 高度なトピック

### 第19-20章: 負荷分散

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| フロントエンド負荷分散 | CloudFront, Route 53 | エッジ/DNS |
| データセンター負荷分散 | ALB, NLB | L7/L4 |
| ヘルスチェック | ELB Health Checks, Route 53 | 正常性確認 |

### 第21章: 過負荷への対処

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| レート制限 | API Gateway Throttling, WAF | リクエスト制限 |
| キューイング | SQS | バッファリング |
| バックプレッシャー | Lambda Concurrency Limits | 同時実行制限 |

### 第22章: カスケード障害への対処

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| サーキットブレーカー | - (アプリ実装) | Hystrix等 |
| タイムアウト | Lambda Timeout, ELB Timeout | 時間制限 |
| リトライ戦略 | Step Functions Retry | 指数バックオフ |
| グレースフルデグラデーション | - (アプリ設計) | 機能縮退 |

### 第26章: データの整合性

| SRE概念 | AWS対応サービス/機能 | 備考 |
|---------|---------------------|------|
| バックアップ | AWS Backup, RDS Snapshots | データ保護 |
| ポイントインタイムリカバリ | RDS PITR, DynamoDB PITR | 時点復旧 |
| レプリケーション | S3 CRR, DynamoDB Global Tables | データ複製 |

---

## 試験対策ポイント

### SRE概念からAWS問題を解くコツ

1. **SLO/SLIが出たら** → CloudWatch メトリクス、アラーム設計
2. **トイル削減が出たら** → SSM Automation、Lambda自動化
3. **インシデント対応が出たら** → Incident Manager、ランブック
4. **カスケード障害が出たら** → タイムアウト、リトライ、サーキットブレーカー
5. **リリースエンジニアリングが出たら** → CodePipeline、デプロイ戦略

---

*最終更新: 2026-01-21*
