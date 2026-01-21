# AWS Auto Scaling

## 概要

AWS Auto Scalingは、アプリケーションの需要に応じてリソースを自動的にスケールするサービス群。EC2 Auto Scaling、Application Auto Scaling、AWS Auto Scaling（統合コンソール）の3つのコンポーネントで構成される。コスト最適化とパフォーマンス維持のバランスを自動化する。

## キーコンセプト

### EC2 Auto Scaling
- **Auto Scalingグループ（ASG）**: EC2インスタンスの論理グループ
- **起動テンプレート/起動設定**: インスタンスの設定を定義
- **希望キャパシティ**: 目標インスタンス数
- **最小/最大キャパシティ**: スケーリングの範囲
- **スケーリングポリシー**: スケーリングのルール
- **ウォームプール**: 事前にウォームアップされたインスタンスプール
- **インスタンスリフレッシュ**: ローリング更新

### Application Auto Scaling
- ECS、DynamoDB、Lambda、Aurora等の非EC2リソースをスケール
- ターゲット追跡、ステップ、スケジュールポリシーをサポート

## 試験頻出ポイント

- [ ] スケーリングポリシーの種類と選択基準
- [ ] ターゲット追跡スケーリングの設定
- [ ] 予測スケーリング（Predictive Scaling）
- [ ] ウォームプールの設定と用途
- [ ] インスタンスリフレッシュの設定
- [ ] ヘルスチェックタイプ（EC2, ELB）
- [ ] 起動テンプレート vs 起動設定
- [ ] スケールイン保護
- [ ] ライフサイクルフック

## DOP-C02での出題パターン

### パターン1: 予測可能な負荷変動
- **シナリオ**: 毎日9時〜18時に負荷が高い
- **ポイント**: スケジュールスケーリング または 予測スケーリング

### パターン2: 安定したパフォーマンス維持
- **シナリオ**: CPU使用率を50%に維持したい
- **ポイント**: ターゲット追跡スケーリング

### パターン3: 起動時間の短縮
- **シナリオ**: スケールアウト時のインスタンス起動が遅い
- **ポイント**: ウォームプール

### パターン4: 段階的な更新
- **シナリオ**: AMIを更新してローリングデプロイしたい
- **ポイント**: インスタンスリフレッシュ

### パターン5: カスタム処理の実行
- **シナリオ**: インスタンス終了前にログを退避
- **ポイント**: ライフサイクルフック

## ベストプラクティス

1. **ターゲット追跡の使用**: シンプルで効果的なスケーリング
2. **複数AZへの分散**: 耐障害性の向上
3. **ELBヘルスチェック**: アプリケーションレベルの健全性確認
4. **ウォームプールの活用**: 起動時間が長いアプリケーション向け
5. **混合インスタンスポリシー**: コスト最適化（スポット + オンデマンド）
6. **スケーリングクールダウン**: 過剰なスケーリングを防止

## スケーリングポリシー

| ポリシー | 説明 | ユースケース |
|----------|------|--------------|
| ターゲット追跡 | 目標値を維持 | CPU 50%維持など |
| ステップスケーリング | 段階的にスケール | 細かい制御が必要な場合 |
| シンプルスケーリング | 単一アクション | 基本的なスケーリング |
| 予測スケーリング | ML予測でプロビジョニング | パターンのあるワークロード |
| スケジュールスケーリング | 時間ベース | 予測可能な負荷変動 |

### ターゲット追跡スケーリング

```yaml
# CloudFormation
ScalingPolicy:
  Type: AWS::AutoScaling::ScalingPolicy
  Properties:
    AutoScalingGroupName: !Ref AutoScalingGroup
    PolicyType: TargetTrackingScaling
    TargetTrackingConfiguration:
      TargetValue: 50.0
      PredefinedMetricSpecification:
        PredefinedMetricType: ASGAverageCPUUtilization
      # または カスタムメトリクス
      # CustomizedMetricSpecification:
      #   MetricName: RequestCountPerTarget
      #   Namespace: MyApp
      #   Statistic: Average
```

### 予測スケーリング

```yaml
PredictiveScalingPolicy:
  Type: AWS::AutoScaling::ScalingPolicy
  Properties:
    AutoScalingGroupName: !Ref AutoScalingGroup
    PolicyType: PredictiveScaling
    PredictiveScalingConfiguration:
      MetricSpecifications:
        - TargetValue: 50
          PredefinedMetricPairSpecification:
            PredefinedMetricType: ASGCPUUtilization
      Mode: ForecastAndScale  # ForecastOnly または ForecastAndScale
      SchedulingBufferTime: 300
```

## ウォームプール

### 用途

- インスタンス起動時間が長い場合
- 急激なスケールアウトに対応
- コールドスタートを回避

### 設定

```yaml
WarmPool:
  Type: AWS::AutoScaling::WarmPool
  Properties:
    AutoScalingGroupName: !Ref AutoScalingGroup
    PoolState: Stopped  # Stopped, Running, Hibernated
    MinSize: 2
    MaxGroupPreparedCapacity: 10
    InstanceReusePolicy:
      ReuseOnScaleIn: true
```

### 状態

| 状態 | 説明 | コスト |
|------|------|--------|
| Stopped | 停止状態で待機 | EBS料金のみ |
| Running | 実行状態で待機 | フルコスト |
| Hibernated | 休止状態で待機 | EBS料金のみ |

## インスタンスリフレッシュ

```yaml
# CLI での実行
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name my-asg \
  --preferences '{
    "MinHealthyPercentage": 90,
    "InstanceWarmup": 300,
    "MaxHealthyPercentage": 110
  }'
```

### 設定オプション

| 設定 | 説明 |
|------|------|
| MinHealthyPercentage | 最小健全インスタンス割合 |
| MaxHealthyPercentage | 最大健全インスタンス割合（オーバープロビジョニング） |
| InstanceWarmup | ウォームアップ時間 |
| CheckpointDelay | チェックポイント間の待機時間 |
| CheckpointPercentages | 段階的な更新割合 |

## ライフサイクルフック

```
スケールアウト時:
Pending → Pending:Wait → Pending:Proceed → InService

スケールイン時:
InService → Terminating:Wait → Terminating:Proceed → Terminated
```

### 設定例

```yaml
TerminationLifecycleHook:
  Type: AWS::AutoScaling::LifecycleHook
  Properties:
    AutoScalingGroupName: !Ref AutoScalingGroup
    LifecycleTransition: autoscaling:EC2_INSTANCE_TERMINATING
    HeartbeatTimeout: 300
    DefaultResult: CONTINUE
    NotificationTargetARN: !Ref HookSNSTopic
    RoleARN: !GetAtt HookRole.Arn
```

### 完了通知

```bash
# Lambda等から完了を通知
aws autoscaling complete-lifecycle-action \
  --lifecycle-hook-name my-hook \
  --auto-scaling-group-name my-asg \
  --lifecycle-action-result CONTINUE \
  --instance-id i-1234567890abcdef0
```

## 混合インスタンスポリシー

```yaml
MixedInstancesPolicy:
  InstancesDistribution:
    OnDemandBaseCapacity: 2
    OnDemandPercentageAboveBaseCapacity: 25
    SpotAllocationStrategy: capacity-optimized
  LaunchTemplate:
    LaunchTemplateSpecification:
      LaunchTemplateId: !Ref LaunchTemplate
      Version: !GetAtt LaunchTemplate.LatestVersionNumber
    Overrides:
      - InstanceType: t3.medium
      - InstanceType: t3.large
      - InstanceType: t3a.medium
```

## ヘルスチェック

| タイプ | 説明 | チェック内容 |
|--------|------|--------------|
| EC2 | デフォルト | EC2インスタンスの状態 |
| ELB | ELB連携時 | ターゲットグループのヘルスチェック |
| VPC Lattice | Lattice連携時 | ターゲットグループのヘルスチェック |

```yaml
AutoScalingGroup:
  Type: AWS::AutoScaling::AutoScalingGroup
  Properties:
    HealthCheckType: ELB
    HealthCheckGracePeriod: 300
```

## Application Auto Scaling

### サポートされるリソース

| サービス | スケール対象 |
|----------|--------------|
| ECS | タスク数 |
| DynamoDB | 読み取り/書き込みキャパシティ |
| Aurora | レプリカ数 |
| Lambda | プロビジョニング済み同時実行数 |
| Comprehend | エンドポイント |
| SageMaker | エンドポイント |

### 設定例（ECS）

```yaml
# スケーラブルターゲット
ScalableTarget:
  Type: AWS::ApplicationAutoScaling::ScalableTarget
  Properties:
    ServiceNamespace: ecs
    ScalableDimension: ecs:service:DesiredCount
    ResourceId: !Sub "service/${Cluster}/${Service.Name}"
    MinCapacity: 2
    MaxCapacity: 10
    RoleARN: !GetAtt AutoScalingRole.Arn

# スケーリングポリシー
ScalingPolicy:
  Type: AWS::ApplicationAutoScaling::ScalingPolicy
  Properties:
    PolicyName: cpu-tracking
    PolicyType: TargetTrackingScaling
    ScalingTargetId: !Ref ScalableTarget
    TargetTrackingScalingPolicyConfiguration:
      TargetValue: 70.0
      PredefinedMetricSpecification:
        PredefinedMetricType: ECSServiceAverageCPUUtilization
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| ELB | ヘルスチェック、トラフィック分散 |
| EC2 | インスタンス管理 |
| CloudWatch | メトリクスベーススケーリング |
| SNS | ライフサイクルフック通知 |
| Lambda | ライフサイクルフック処理 |
| EventBridge | スケーリングイベント |
| Systems Manager | パラメータストア連携 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [EC2 Auto Scaling ユーザーガイド](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)
- [Application Auto Scaling ユーザーガイド](https://docs.aws.amazon.com/autoscaling/application/userguide/)
- [スケーリングポリシー](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html)
- [ウォームプール](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-warm-pools.html)
