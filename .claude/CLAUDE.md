# AWS DevOps Professional 学習プロジェクト

## プロジェクト概要

AWS DevOps Professional (DOP-C02) 資格取得に向けた学習管理リポジトリ。

## Claude への指示

### 学習支援時のルール

1. **試験範囲に沿った回答**: 回答はDOP-C02の試験範囲に沿った内容にする
2. **実践的な例**: 可能な限り実際のユースケースやハンズオン手順を含める
3. **ベストプラクティス**: AWSのベストプラクティスを踏まえた回答
4. **SRE本との連携**: 関連するSRE概念があれば言及する

### サービス学習ファイルの形式

新しいサービス学習ファイルを作成する際は以下のテンプレートに従う:

```markdown
# [サービス名]

## 概要
[サービスの目的・位置づけ]

## キーコンセプト
- ポイント1
- ポイント2

## 試験頻出ポイント
- [ ] トピック1
- [ ] トピック2

## SRE本との関連
- 第X章: [章タイトル]

## ハンズオン記録
[実際に構築した内容・気づき]

## 参考リンク
- [公式ドキュメント](URL)
```

### ハンズオン記録の形式

```markdown
# ハンズオン: [タイトル]

## 目的
[このハンズオンで学ぶこと]

## 前提条件
- AWS アカウント
- 必要な権限

## 手順

### Step 1: [ステップ名]
[手順の説明]

### Step 2: [ステップ名]
[手順の説明]

## 学んだこと
- ポイント1
- ポイント2

## トラブルシューティング
[発生した問題と解決策]

---
*実施日: YYYY-MM-DD*
```

### 用語集

| 英語 | 日本語 | 備考 |
|------|--------|------|
| CI/CD | 継続的インテグレーション/継続的デリバリー | |
| IaC | Infrastructure as Code | |
| Blue/Green Deployment | Blue/Greenデプロイ | |
| Canary Deployment | カナリアデプロイ | |
| Rolling Update | ローリングアップデート | |
| Immutable Infrastructure | イミュータブルインフラ | |
| GitOps | GitOps | |
| Drift Detection | ドリフト検出 | |
| Change Set | 変更セット | CloudFormation |
| Stack | スタック | CloudFormation |
| Nested Stack | ネストスタック | CloudFormation |
| Cross-Stack Reference | クロススタック参照 | |
| Custom Resource | カスタムリソース | |
| State Machine | ステートマシン | Step Functions |
| Runbook | ランブック | |
| Automation Document | オートメーションドキュメント | SSM |
| Parameter Store | パラメータストア | SSM |
| Secrets Manager | Secrets Manager | |
| Service Control Policy | サービスコントロールポリシー (SCP) | |
| Organizational Unit | 組織単位 (OU) | |
| Guard Duty | GuardDuty | |
| Config Rule | Configルール | |
| Remediation | 修復 | AWS Config |
| Composite Alarm | 複合アラーム | CloudWatch |
| Metric Filter | メトリクスフィルター | CloudWatch Logs |
| Subscription Filter | サブスクリプションフィルター | CloudWatch Logs |
| Trace | トレース | X-Ray |
| Segment | セグメント | X-Ray |
| Service Map | サービスマップ | X-Ray |
| Health Check | ヘルスチェック | Route 53, ELB |
| Failover | フェイルオーバー | |
| Target Group | ターゲットグループ | ELB |
| Launch Template | 起動テンプレート | |
| Scaling Policy | スケーリングポリシー | |
| Warm Pool | ウォームプール | Auto Scaling |
| Global Table | グローバルテーブル | DynamoDB |
| Read Replica | リードレプリカ | RDS |
| Multi-AZ | マルチAZ | |
| Cross-Region | クロスリージョン | |

## ディレクトリ構成

```
aws-dop-study/
├── domain-1-sdlc-automation/    # SDLC自動化 (22%)
├── domain-2-config-management/  # 構成管理・IaC (17%)
├── domain-3-monitoring-logging/ # 監視・ロギング (15%)
├── domain-4-policies-standards/ # ポリシー・標準 (10%)
├── domain-5-incident-event/     # インシデント・イベント対応 (18%)
├── domain-6-ha-fault-tolerance/ # 高可用性・耐障害性 (18%)
├── exam-prep/                   # 試験対策
├── sre-aws-mapping/             # SRE本との連携
└── resources/                   # 参考リソース
```

## 進捗確認

`PROGRESS.md` を参照して現在の学習状況を確認すること。
