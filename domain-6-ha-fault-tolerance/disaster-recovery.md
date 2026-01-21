# Disaster Recovery (DR) 戦略

## 概要

災害復旧（Disaster Recovery）は、災害や障害発生時にビジネスを継続するための計画と実装。AWSでは、複数のリージョンとサービスを組み合わせて、コストとRTO/RPOのバランスを取りながらDR戦略を構築できる。

## キーコンセプト

- **RTO（Recovery Time Objective）**: 復旧目標時間 - 障害発生からサービス復旧までの許容時間
- **RPO（Recovery Point Objective）**: 復旧目標時点 - 許容されるデータ損失の時間範囲
- **フェイルオーバー**: プライマリからDRサイトへの切り替え
- **フェイルバック**: DRサイトからプライマリへの復帰
- **パイロットライト**: 最小限のコア機能を常時稼働
- **ウォームスタンバイ**: 縮小版の完全環境を維持

## DR戦略の比較

| 戦略 | RTO | RPO | コスト | 複雑さ |
|------|-----|-----|--------|--------|
| Backup & Restore | 時間〜日 | 最大24時間 | 最低 | 低 |
| Pilot Light | 10分〜時間 | 分 | 低 | 中 |
| Warm Standby | 分 | 秒〜分 | 中 | 中〜高 |
| Multi-Site Active-Active | 秒〜ゼロ | ゼロ | 最高 | 高 |

## 試験頻出ポイント

- [ ] DR戦略の選択基準（RTO/RPO要件とコスト）
- [ ] 各戦略の構成要素と実装方法
- [ ] クロスリージョンレプリケーション（S3, RDS, DynamoDB）
- [ ] Route 53フェイルオーバーの設定
- [ ] AWS Backupによる一元管理
- [ ] パイロットライトとウォームスタンバイの違い
- [ ] フェイルオーバー/フェイルバックの手順
- [ ] Resilience Hubによる評価

## DOP-C02での出題パターン

### パターン1: コスト重視のDR
- **シナリオ**: 予算が限られているがDRは必要
- **ポイント**: Backup & Restore または Pilot Light

### パターン2: 低RTO要件
- **シナリオ**: 5分以内に復旧が必要
- **ポイント**: Warm Standby または Active-Active

### パターン3: データ整合性要件
- **シナリオ**: データ損失が許容されない
- **ポイント**: 同期レプリケーション、Active-Active

### パターン4: 自動フェイルオーバー
- **シナリオ**: 人手を介さずに自動で切り替えたい
- **ポイント**: Route 53ヘルスチェック + フェイルオーバー

## 各DR戦略の詳細

### 1. Backup & Restore

**特徴**:
- 最もシンプルで低コスト
- データのバックアップのみを別リージョンに保存
- 復旧時にインフラを一から構築

**実装**:
```
プライマリリージョン          DRリージョン
┌─────────────────┐      ┌─────────────────┐
│ EC2, RDS, etc.  │      │    S3 Backup    │
│                 │ ───→ │    AMI Copy     │
│ 通常運用        │      │                 │
└─────────────────┘      └─────────────────┘
```

**構成要素**:
- S3クロスリージョンレプリケーション
- AMIのコピー
- RDSスナップショットのコピー
- AWS Backupによる一元管理
- CloudFormation/CDKテンプレート

### 2. Pilot Light

**特徴**:
- 最小限のコアコンポーネントを常時稼働
- データベースのみレプリケーション
- 障害時にスケールアップ

**実装**:
```
プライマリリージョン          DRリージョン
┌─────────────────┐      ┌─────────────────┐
│ EC2 (Full)      │      │ EC2 (Stopped)   │
│ RDS (Primary)   │ ───→ │ RDS (Replica)   │
│ ELB (Active)    │      │ ELB (Inactive)  │
└─────────────────┘      └─────────────────┘
```

**構成要素**:
- RDSクロスリージョンリードレプリカ
- Aurora Global Database
- 最小限のEC2インスタンス（停止状態）
- Route 53フェイルオーバー設定

### 3. Warm Standby

**特徴**:
- 縮小版の完全な環境を維持
- 常時稼働しているが最小キャパシティ
- 障害時に即座にスケールアップ

**実装**:
```
プライマリリージョン          DRリージョン
┌─────────────────┐      ┌─────────────────┐
│ EC2 (10台)      │      │ EC2 (2台)       │
│ RDS (Primary)   │ ───→ │ RDS (Replica)   │
│ ELB (100%)      │      │ ELB (10%)       │
└─────────────────┘      └─────────────────┘
```

**構成要素**:
- 縮小版のAuto Scalingグループ
- RDSリードレプリカ（プロモーション可能）
- ELB + 最小限のターゲット
- Route 53加重ルーティング

### 4. Multi-Site Active-Active

**特徴**:
- 両リージョンがアクティブに稼働
- トラフィックを分散
- 即時フェイルオーバー

**実装**:
```
プライマリリージョン          セカンダリリージョン
┌─────────────────┐      ┌─────────────────┐
│ EC2 (Full)      │      │ EC2 (Full)      │
│ Aurora (Writer) │ ←──→ │ Aurora (Writer) │
│ ELB (50%)       │      │ ELB (50%)       │
└─────────────────┘      └─────────────────┘
         ↑                       ↑
         └───── Route 53 ────────┘
              (レイテンシ or 加重)
```

**構成要素**:
- Aurora Global Database（書き込み転送）
- DynamoDB Global Tables
- Route 53レイテンシベースルーティング
- CloudFront + 複数オリジン

## クロスリージョンレプリケーション

### S3

```yaml
ReplicationConfiguration:
  Role: !GetAtt ReplicationRole.Arn
  Rules:
    - Status: Enabled
      Destination:
        Bucket: arn:aws:s3:::dr-bucket
        ReplicationTime:
          Status: Enabled
          Time:
            Minutes: 15
        Metrics:
          Status: Enabled
```

### RDS

```yaml
# クロスリージョンリードレプリカ
ReadReplica:
  Type: AWS::RDS::DBInstance
  Properties:
    SourceDBInstanceIdentifier: !Sub "arn:aws:rds:us-east-1:${AWS::AccountId}:db:primary"
    DBInstanceClass: db.r5.large
    AvailabilityZone: us-west-2a
```

### DynamoDB Global Tables

```yaml
GlobalTable:
  Type: AWS::DynamoDB::GlobalTable
  Properties:
    TableName: my-global-table
    Replicas:
      - Region: us-east-1
      - Region: us-west-2
      - Region: eu-west-1
```

### Aurora Global Database

```yaml
GlobalCluster:
  Type: AWS::RDS::GlobalCluster
  Properties:
    GlobalClusterIdentifier: my-global-cluster
    SourceDBClusterIdentifier: !Ref PrimaryCluster
```

## AWS Backup

### 一元管理

```yaml
BackupPlan:
  Type: AWS::Backup::BackupPlan
  Properties:
    BackupPlan:
      BackupPlanName: cross-region-backup
      BackupPlanRule:
        - RuleName: daily-backup
          TargetBackupVault: !Ref BackupVault
          ScheduleExpression: cron(0 5 * * ? *)
          StartWindowMinutes: 60
          CompletionWindowMinutes: 120
          Lifecycle:
            DeleteAfterDays: 30
          CopyActions:
            - DestinationBackupVaultArn: arn:aws:backup:us-west-2:*:backup-vault:dr-vault
              Lifecycle:
                DeleteAfterDays: 90
```

## Resilience Hub

### 機能

- アプリケーションの回復力を評価
- RTO/RPO目標との差分を特定
- 改善推奨事項を提供
- 運用手順（ランブック）を自動生成

### 評価項目

- インフラストラクチャ
- AZ障害
- リージョン障害
- アプリケーション障害

## フェイルオーバー手順

### 自動フェイルオーバー

1. Route 53ヘルスチェックがプライマリの異常を検知
2. DNSがセカンダリにフェイルオーバー
3. トラフィックがDRリージョンへ

### 手動フェイルオーバー（Pilot Light/Warm Standby）

1. RDSリードレプリカをプロモート
2. EC2インスタンスを起動/スケールアップ
3. Route 53レコードを更新
4. 動作確認

## 他サービスとの連携

| サービス | DR用途 |
|----------|--------|
| Route 53 | フェイルオーバールーティング |
| S3 | クロスリージョンレプリケーション |
| RDS/Aurora | クロスリージョンリードレプリカ |
| DynamoDB | Global Tables |
| EBS | スナップショットコピー |
| AWS Backup | 一元的なバックアップ管理 |
| CloudFormation | インフラの再構築 |
| Resilience Hub | 回復力の評価 |
| Elastic Disaster Recovery | サーバーレベルのDR |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS DR ホワイトペーパー](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)
- [AWS Backup ユーザーガイド](https://docs.aws.amazon.com/aws-backup/latest/devguide/)
- [Resilience Hub ユーザーガイド](https://docs.aws.amazon.com/resilience-hub/latest/userguide/)
- [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)
