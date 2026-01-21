# AWS CodeDeploy

## 概要

AWS CodeDeployは、EC2インスタンス、オンプレミスサーバー、Lambda関数、ECSサービスへのアプリケーションデプロイを自動化するフルマネージドのデプロイサービス。ダウンタイムを最小化しながら、新機能のリリースを迅速かつ確実に行える。

## キーコンセプト

- **アプリケーション**: デプロイ対象のコンテナ（EC2/オンプレミス、Lambda、ECS）
- **デプロイグループ**: デプロイ先のインスタンス/サービスのグループ
- **デプロイ設定**: デプロイの速度とロールバック条件
- **AppSpec.yml**: デプロイ手順を定義するファイル
- **リビジョン**: アプリケーションの特定バージョン（S3またはGitHub）
- **デプロイタイプ**: In-Place（EC2/オンプレミス）またはBlue/Green
- **ライフサイクルイベント**: デプロイ中の各フェーズでのフック

## 試験頻出ポイント

- [ ] デプロイタイプの違い（In-Place vs Blue/Green）
- [ ] コンピューティングプラットフォーム別の機能差異（EC2、Lambda、ECS）
- [ ] AppSpec.ymlの構造とライフサイクルイベント
- [ ] デプロイ設定（OneAtATime、HalfAtATime、AllAtOnce、カスタム）
- [ ] 自動ロールバックの条件設定
- [ ] トラフィックシフト戦略（Linear、Canary、AllAtOnce）
- [ ] CodeDeployエージェントの役割と管理
- [ ] ロードバランサーとの連携

## DOP-C02での出題パターン

### パターン1: ゼロダウンタイムデプロイ
- **シナリオ**: 本番環境でサービス停止なしにデプロイしたい
- **ポイント**: Blue/Greenデプロイ + ALB/ELB連携

### パターン2: 段階的ロールアウト
- **シナリオ**: 新バージョンを一部のトラフィックで検証してから全体展開
- **ポイント**: Canaryデプロイ設定（Lambda/ECS）

### パターン3: 自動ロールバック
- **シナリオ**: デプロイ後にCloudWatchアラームがトリガーされたら自動ロールバック
- **ポイント**: デプロイグループでアラームベースのロールバック設定

### パターン4: ハイブリッド環境へのデプロイ
- **シナリオ**: EC2とオンプレミスサーバーに同時デプロイ
- **ポイント**: オンプレミスインスタンスの登録、タグベースのデプロイグループ

## ベストプラクティス

1. **Blue/Greenデプロイの活用**: 本番環境ではゼロダウンタイムデプロイを推奨
2. **自動ロールバックの設定**: CloudWatchアラームと連携して異常時に自動ロールバック
3. **段階的デプロイ**: CanaryまたはLinear設定でリスクを低減
4. **ヘルスチェックの活用**: ライフサイクルフックでヘルスチェックを実行
5. **エージェントの自動更新**: SSM State Managerでエージェントを最新に維持
6. **タグベースのデプロイグループ**: インスタンスタグでデプロイ対象を管理

## デプロイタイプ比較

| 項目 | In-Place | Blue/Green |
|------|----------|------------|
| 対象 | EC2/オンプレミスのみ | EC2, Lambda, ECS |
| ダウンタイム | あり（インスタンス単位） | なし |
| ロールバック | 再デプロイ必要 | 即時切り替え |
| コスト | 低い | 一時的に2倍のリソース |
| ユースケース | 開発/ステージング | 本番環境 |

## コンピューティングプラットフォーム別機能

| 機能 | EC2/オンプレミス | Lambda | ECS |
|------|------------------|--------|-----|
| In-Place | ✓ | - | - |
| Blue/Green | ✓ | ✓ | ✓ |
| Canary | - | ✓ | ✓ |
| Linear | - | ✓ | ✓ |
| ロールバック | 再デプロイ | 即時 | 即時 |

## AppSpec.yml構造

### EC2/オンプレミス

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  BeforeInstall:
    - location: scripts/stop_server.sh
      timeout: 300
  AfterInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
  ValidateService:
    - location: scripts/health_check.sh
      timeout: 300
```

### Lambda

```yaml
version: 0.0
Resources:
  - MyFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: MyFunction
        Alias: live
        CurrentVersion: 1
        TargetVersion: 2
Hooks:
  - BeforeAllowTraffic: PreTrafficHookFunction
  - AfterAllowTraffic: PostTrafficHookFunction
```

### ECS

```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "arn:aws:ecs:..."
        LoadBalancerInfo:
          ContainerName: "my-container"
          ContainerPort: 80
Hooks:
  - BeforeInstall: "BeforeInstallHookFunction"
  - AfterInstall: "AfterInstallHookFunction"
  - AfterAllowTestTraffic: "AfterAllowTestTrafficHookFunction"
  - BeforeAllowTraffic: "BeforeAllowTrafficHookFunction"
  - AfterAllowTraffic: "AfterAllowTrafficHookFunction"
```

## ライフサイクルイベント（EC2/オンプレミス）

| イベント | タイミング | 用途 |
|----------|------------|------|
| ApplicationStop | デプロイ開始前 | 既存アプリの停止 |
| DownloadBundle | - | アプリのダウンロード（自動） |
| BeforeInstall | インストール前 | ファイルの復号化、バックアップ |
| Install | - | ファイルのコピー（自動） |
| AfterInstall | インストール後 | 設定変更、権限設定 |
| ApplicationStart | アプリ起動 | サービス開始 |
| ValidateService | 検証 | ヘルスチェック |

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| CodePipeline | デプロイステージとして統合 |
| S3 | リビジョンの保存 |
| EC2 Auto Scaling | スケールアウト時の自動デプロイ |
| ALB/NLB | トラフィック切り替え |
| CloudWatch Alarms | 自動ロールバックのトリガー |
| Lambda | デプロイフック、サーバーレスデプロイ |
| ECS | コンテナデプロイ |
| Systems Manager | エージェント管理 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS CodeDeploy ユーザーガイド](https://docs.aws.amazon.com/codedeploy/latest/userguide/)
- [AppSpec ファイルリファレンス](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)
- [デプロイ設定](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html)
