# AWS DevOps Professional (DOP-C02) 学習リポジトリ

> AWS DevOps Professional 資格取得に向けた学習管理・知識整理用リポジトリ

## 試験概要

| 項目 | 内容 |
|------|------|
| 試験コード | DOP-C02 |
| 問題数 | 75問（多肢選択・複数回答） |
| 試験時間 | 180分 |
| 合格スコア | 750/1000 |
| 受験料 | $300 USD |
| 推奨経験 | 2年以上のAWS環境でのDevOps経験 |

## ドメイン構成

| ドメイン | 内容 | 配点 |
|----------|------|------|
| 1 | SDLC自動化 | 22% |
| 2 | 構成管理・IaC | 17% |
| 3 | 監視・ロギング | 15% |
| 4 | ポリシー・標準 | 10% |
| 5 | インシデント・イベント対応 | 18% |
| 6 | 高可用性・耐障害性 | 18% |

## リポジトリ構成

```
aws-dop-study/
├── README.md                    # このファイル
├── PROGRESS.md                  # 学習進捗管理
├── CLAUDE.md                    # Claude Code設定
│
├── domain-1-sdlc-automation/    # ドメイン1: SDLC自動化
├── domain-2-config-management/  # ドメイン2: 構成管理・IaC
├── domain-3-monitoring-logging/ # ドメイン3: 監視・ロギング
├── domain-4-policies-standards/ # ドメイン4: ポリシー・標準
├── domain-5-incident-event/     # ドメイン5: インシデント・イベント対応
├── domain-6-ha-fault-tolerance/ # ドメイン6: 高可用性・耐障害性
│
├── exam-prep/                   # 試験対策
├── sre-aws-mapping/             # SRE本との知識連携
└── resources/                   # 参考リソース
```

## 学習方針

### バランス型アプローチ

1. **理論学習**: 各サービスの概念・ベストプラクティスを理解
2. **ハンズオン**: 実際にAWSで構築・検証
3. **模擬試験**: 定期的に模試を受けて理解度を確認
4. **振り返り**: 間違えた問題を分析・深堀り

### 学習サイクル

```
理論学習 → ハンズオン → 模試 → 振り返り → 理論学習...
```

## SRE本との連携

現在翻訳中の[SRE本](../sre-book/)の知識を活用し、DevOps概念を深く理解する。

| SRE概念 | AWS関連サービス |
|---------|-----------------|
| SLO/SLI | CloudWatch メトリクス、Synthetics |
| トイル削減 | Systems Manager Automation、Lambda |
| インシデント対応 | EventBridge、Incident Manager |
| ポストモーテム | CloudWatch Logs Insights |
| リリースエンジニアリング | CodePipeline、Blue/Greenデプロイ |
| モニタリング | CloudWatch、X-Ray、CloudTrail |

## 進捗確認

[PROGRESS.md](./PROGRESS.md) を参照

## 参考リソース

- [AWS 公式試験ガイド](https://aws.amazon.com/certification/certified-devops-engineer-professional/)
- [AWS Skill Builder](https://explore.skillbuilder.aws/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

*作成日: 2026-01-21*
