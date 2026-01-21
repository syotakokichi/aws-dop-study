# Amazon CloudWatch

## 概要

Amazon CloudWatchは、AWSリソースとアプリケーションのモニタリングと可観測性を提供するサービス。メトリクス収集、ログ管理、アラーム設定、ダッシュボード作成などの機能を通じて、運用の可視化と自動化を実現する。

## キーコンセプト

- **メトリクス**: 時系列データポイント（CPU使用率、レイテンシ等）
- **名前空間（Namespace）**: メトリクスをグループ化するコンテナ
- **ディメンション**: メトリクスを識別する名前/値のペア
- **統計（Statistics）**: Average, Sum, Minimum, Maximum, SampleCount, Percentile
- **期間（Period）**: メトリクスを集約する時間間隔
- **アラーム**: メトリクス値に基づくアクション実行
- **ダッシュボード**: メトリクスの可視化
- **カスタムメトリクス**: アプリケーション固有のメトリクスを送信

## 試験頻出ポイント

- [ ] 標準メトリクスとカスタムメトリクスの違い
- [ ] メトリクスの解像度（標準: 1分、高解像度: 1秒）
- [ ] メトリクスの保持期間
- [ ] アラームの状態（OK, ALARM, INSUFFICIENT_DATA）
- [ ] 複合アラーム（Composite Alarm）
- [ ] アラームアクション（EC2, Auto Scaling, SNS, Systems Manager）
- [ ] CloudWatch Agent の役割と設定
- [ ] 名前空間とディメンションの設計

## DOP-C02での出題パターン

### パターン1: カスタムメトリクスの実装
- **シナリオ**: アプリケーション固有のビジネスメトリクスを監視したい
- **ポイント**: CloudWatch Agent または PutMetricData API

### パターン2: 高解像度メトリクス
- **シナリオ**: 1秒単位でメトリクスを収集したい
- **ポイント**: 高解像度カスタムメトリクス（StorageResolution: 1）

### パターン3: 複合アラーム
- **シナリオ**: 複数の条件を組み合わせてアラートを発生させたい
- **ポイント**: Composite Alarm（AND/OR/NOT演算子）

### パターン4: 自動修復
- **シナリオ**: アラーム発生時に自動でEC2を再起動
- **ポイント**: アラームアクション（EC2 Recover/Reboot）

## ベストプラクティス

1. **名前空間の設計**: アプリケーション/環境で整理
2. **ディメンションの活用**: インスタンスID、環境名などで分類
3. **アラームしきい値の調整**: 誤検知を防ぐため適切な値を設定
4. **複合アラームの活用**: ノイズを削減し、本当の問題に集中
5. **ダッシュボードの作成**: 重要メトリクスを一覧表示
6. **コスト最適化**: 不要なカスタムメトリクスを削除

## メトリクスの解像度と保持期間

| 解像度 | 保持期間 |
|--------|----------|
| 1秒（高解像度） | 3時間 |
| 60秒（1分） | 15日 |
| 300秒（5分） | 63日 |
| 3600秒（1時間） | 455日（15ヶ月） |

**注意**: データは自動的に集約される（1分データは15日後に5分データに集約）

## 標準メトリクス vs カスタムメトリクス

| 項目 | 標準メトリクス | カスタムメトリクス |
|------|----------------|-------------------|
| 送信方法 | AWSが自動送信 | API/Agent経由 |
| 解像度 | 1分または5分 | 1秒〜（高解像度） |
| 料金 | 無料（多くの場合） | 有料 |
| 例 | EC2 CPUUtilization | アプリケーションエラー数 |

## アラーム状態

| 状態 | 説明 |
|------|------|
| OK | しきい値内 |
| ALARM | しきい値超過 |
| INSUFFICIENT_DATA | データ不足（起動直後など） |

## アラームアクション

| アクション種類 | 説明 |
|----------------|------|
| SNS通知 | メール、SMS、Lambda起動等 |
| Auto Scaling | スケールアウト/イン |
| EC2アクション | Stop, Terminate, Reboot, Recover |
| Systems Manager | Automation Runbook実行 |

## 複合アラーム（Composite Alarm）

```
# 例: CPUアラームとメモリアラームの両方がALARM状態の時のみ通知
ALARM(CPUAlarm) AND ALARM(MemoryAlarm)

# 例: いずれかのアラームがALARM状態の時に通知
ALARM(CPUAlarm) OR ALARM(MemoryAlarm)

# 例: メンテナンス中はアラームを抑制
ALARM(CPUAlarm) AND NOT ALARM(MaintenanceAlarm)
```

## CloudWatch Agent

### 収集可能なデータ

- **メトリクス**: メモリ使用率、ディスク使用率、プロセス情報等
- **ログ**: アプリケーションログ、システムログ

### 設定ファイル構造

```json
{
  "agent": {
    "metrics_collection_interval": 60
  },
  "metrics": {
    "namespace": "MyApp",
    "metrics_collected": {
      "mem": {
        "measurement": ["mem_used_percent"]
      },
      "disk": {
        "measurement": ["used_percent"],
        "resources": ["/", "/data"]
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/myapp/*.log",
            "log_group_name": "myapp-logs"
          }
        ]
      }
    }
  }
}
```

## PutMetricData API

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'RequestCount',
            'Dimensions': [
                {'Name': 'Environment', 'Value': 'Production'},
                {'Name': 'Service', 'Value': 'API'}
            ],
            'Value': 1,
            'Unit': 'Count',
            'StorageResolution': 1  # 高解像度（1秒）
        }
    ]
)
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| EC2 | 標準メトリクス、アラームアクション |
| Auto Scaling | メトリクスベースのスケーリング |
| SNS | アラーム通知 |
| Lambda | カスタムメトリクス送信、アラームトリガー |
| EventBridge | アラーム状態変更イベント |
| Systems Manager | 自動修復アクション |
| X-Ray | トレースとの相関 |
| CloudWatch Logs | ログからメトリクス抽出 |

## ダッシュボード

### 特徴

- **クロスアカウント**: 複数アカウントのメトリクスを表示
- **クロスリージョン**: 複数リージョンのメトリクスを表示
- **自動リフレッシュ**: 定期的に更新
- **共有**: URLまたはスナップショットで共有

### ウィジェットタイプ

| タイプ | 用途 |
|--------|------|
| Line | 時系列データの推移 |
| Stacked area | 積み上げグラフ |
| Number | 最新値の表示 |
| Gauge | ゲージ表示 |
| Text | 説明文 |
| Logs table | ログ表示 |
| Alarm status | アラーム状態 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [Amazon CloudWatch ユーザーガイド](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)
- [CloudWatch Agent 設定](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html)
- [CloudWatch アラーム](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
