# AWS Systems Manager (SSM)

## 概要

AWS Systems Managerは、AWSとオンプレミスのインフラストラクチャを一元的に管理・運用するためのサービス群。パッチ適用、構成管理、オートメーション、パラメータ管理など、運用タスクを自動化・効率化する多数の機能を提供する。

## キーコンセプト

### 主要機能カテゴリ

- **オペレーション管理**: Explorer, OpsCenter, Incident Manager
- **アプリケーション管理**: Parameter Store, AppConfig
- **変更管理**: Automation, Change Manager, Maintenance Windows
- **ノード管理**: Fleet Manager, Session Manager, Patch Manager, State Manager
- **共有リソース**: Documents (SSM Documents)

## 試験頻出ポイント

- [ ] Parameter Storeの階層構造とSecrets Managerとの違い
- [ ] Automationドキュメントによる自動化
- [ ] Run Commandでの一括コマンド実行
- [ ] Session Managerによるセキュアなアクセス（SSHレス）
- [ ] Patch Managerによるパッチ適用
- [ ] State Managerによる構成維持
- [ ] Maintenance Windowsによるスケジュール実行
- [ ] SSMエージェントの役割と前提条件

## DOP-C02での出題パターン

### パターン1: シークレット管理
- **シナリオ**: アプリケーション設定とシークレットを安全に管理
- **ポイント**: Parameter Store（SecureString）vs Secrets Manager

### パターン2: インスタンスへのセキュアアクセス
- **シナリオ**: SSHポートを開けずにEC2に接続したい
- **ポイント**: Session Manager（IAM認証、監査ログ）

### パターン3: パッチ適用の自動化
- **シナリオ**: 数百台のEC2に定期的にパッチを適用
- **ポイント**: Patch Manager + Maintenance Windows

### パターン4: 構成のドリフト修正
- **シナリオ**: インスタンスの構成が期待と異なる場合に自動修正
- **ポイント**: State Manager + Association

### パターン5: 自動修復の実装
- **シナリオ**: CloudWatch Alarmからの自動修復アクション
- **ポイント**: Automation Runbook

## ベストプラクティス

1. **Parameter Storeの階層化**: `/app/env/param`形式で整理
2. **Session Managerの活用**: SSH鍵管理を排除
3. **Maintenance Windowsの設定**: 本番環境の変更は計画的に実行
4. **Automationの活用**: 繰り返しタスクを自動化
5. **IAMの最小権限**: 必要な操作のみ許可
6. **ログの有効化**: Session Managerのログを監査用に保存

## Parameter Store

### 階層と構造

```
/myapp/
  /dev/
    /db/
      host
      password (SecureString)
  /prod/
    /db/
      host
      password (SecureString)
```

### Parameter Storeのティア

| ティア | パラメータ数 | 値サイズ | スループット | 料金 |
|--------|-------------|---------|-------------|------|
| Standard | 10,000 | 4KB | 1,000/秒 | 無料 |
| Advanced | 100,000 | 8KB | 増加可能 | 有料 |

### Parameter Store vs Secrets Manager

| 項目 | Parameter Store | Secrets Manager |
|------|-----------------|-----------------|
| 自動ローテーション | ❌ | ✓ |
| RDS連携 | ❌ | ✓（Lambda自動生成） |
| 料金 | 無料（Standard） | 有料 |
| 値サイズ | 4-8KB | 64KB |
| バージョン管理 | ✓ | ✓ |
| 暗号化 | KMS（SecureString） | KMS |
| クロスアカウント | ❌ | ✓ |

## Session Manager

### 特徴

- **SSHレス**: 22番ポートを開放不要
- **IAM認証**: SSH鍵管理が不要
- **監査ログ**: S3/CloudWatch Logsへの記録
- **ポートフォワーディング**: RDSなどへのトンネリング
- **オンプレミス対応**: ハイブリッド環境もサポート

### 前提条件

1. SSMエージェントのインストール（Amazon Linux 2/Windows Server は標準搭載）
2. IAMロール（AmazonSSMManagedInstanceCore）
3. Systems Managerエンドポイントへの接続（VPCエンドポイント推奨）

## Patch Manager

### コンポーネント

- **パッチベースライン**: 適用するパッチの定義（OSごとに設定）
- **パッチグループ**: タグでインスタンスをグループ化
- **Maintenance Windows**: パッチ適用のスケジュール
- **コンプライアンスレポート**: パッチ適用状況の可視化

### パッチ適用フロー

1. パッチベースラインを定義（承認済みパッチ/拒否パッチ）
2. インスタンスをパッチグループに割り当て（タグ）
3. Maintenance Windowsでスケジュール設定
4. Run CommandまたはAutomationで実行

## State Manager

### 用途

- **構成の維持**: 期待する状態を定義し、ドリフトを自動修正
- **定期実行**: スケジュールで繰り返し適用
- **スケールでの一貫性**: 全インスタンスに同じ構成を適用

### Association

```yaml
# SSM Document + ターゲット + スケジュール
Association:
  Name: AWS-ConfigureCloudWatch
  Targets:
    - Key: tag:Environment
      Values:
        - Production
  ScheduleExpression: rate(30 minutes)
```

## Automation

### 事前定義されたRunbook例

| Runbook | 用途 |
|---------|------|
| AWS-RestartEC2Instance | EC2の再起動 |
| AWS-StopEC2Instance | EC2の停止 |
| AWS-UpdateLinuxAmi | AMIの更新 |
| AWS-CreateSnapshot | EBSスナップショット作成 |
| AWS-PatchInstanceWithRollback | パッチ適用（ロールバック付き） |

### カスタムRunbook

```yaml
schemaVersion: '0.3'
description: Restart EC2 instance
mainSteps:
  - name: stopInstance
    action: aws:changeInstanceState
    inputs:
      InstanceIds:
        - '{{ InstanceId }}'
      DesiredState: stopped
  - name: startInstance
    action: aws:changeInstanceState
    inputs:
      InstanceIds:
        - '{{ InstanceId }}'
      DesiredState: running
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| EC2 | マネージドインスタンスとして管理 |
| CloudWatch | メトリクス/ログ/アラーム連携 |
| EventBridge | イベント駆動でAutomation実行 |
| Lambda | Parameter Store/Secrets Manager参照 |
| CloudFormation | パラメータの動的参照 |
| Config | コンプライアンスチェック |
| IAM | アクセス制御 |
| KMS | パラメータの暗号化 |
| S3 | セッションログ保存 |

## SSM Document タイプ

| タイプ | 用途 |
|--------|------|
| Command | Run Commandで実行 |
| Automation | Automationで実行（複数ステップ） |
| Session | Session Managerの設定 |
| Package | Distributorパッケージ |
| Policy | State Managerのポリシー |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS Systems Manager ユーザーガイド](https://docs.aws.amazon.com/systems-manager/latest/userguide/)
- [Parameter Store ユーザーガイド](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [Session Manager ユーザーガイド](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [Automation ユーザーガイド](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html)
