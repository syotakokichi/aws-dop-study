# AWS X-Ray

## 概要

AWS X-Rayは、分散アプリケーションの分析とデバッグを行うためのサービス。リクエストのトレースを通じて、アプリケーション内のサービス間の依存関係を可視化し、パフォーマンスのボトルネックやエラーの原因を特定できる。

## キーコンセプト

- **トレース（Trace）**: 1つのリクエストが通過するすべてのサービスの記録
- **セグメント（Segment）**: サービスが処理した作業単位
- **サブセグメント（Subsegment）**: セグメント内の詳細な作業単位（外部呼び出し等）
- **サービスマップ**: アプリケーションのアーキテクチャを視覚化
- **サンプリングルール**: トレースするリクエストの割合を制御
- **アノテーション**: トレースをフィルタリングするためのキー/値ペア（インデックス化）
- **メタデータ**: トレースに追加情報を付与（インデックス化なし）
- **グループ**: フィルター式でトレースをグループ化

## 試験頻出ポイント

- [ ] X-Ray SDKとX-Rayデーモンの役割
- [ ] サンプリングルールの設定と動作
- [ ] アノテーションとメタデータの違い
- [ ] サービスマップの読み方
- [ ] Lambda、API Gateway、ECSとの統合
- [ ] クロスアカウントトレース
- [ ] セグメントとサブセグメントの構造
- [ ] トレースヘッダー（X-Amzn-Trace-Id）

## DOP-C02での出題パターン

### パターン1: パフォーマンス問題の特定
- **シナリオ**: マイクロサービスのレイテンシが高い原因を特定したい
- **ポイント**: サービスマップでボトルネックを特定

### パターン2: エラー追跡
- **シナリオ**: 特定のユーザーリクエストがエラーになった原因を調査
- **ポイント**: アノテーションでユーザーIDを記録、トレースでエラーを追跡

### パターン3: サンプリング最適化
- **シナリオ**: トレースのコストを最適化しながら重要なリクエストを記録
- **ポイント**: サンプリングルールの設定（リザーバー、レート）

### パターン4: マルチアカウント監視
- **シナリオ**: 複数アカウントのサービスを一元的にトレース
- **ポイント**: クロスアカウントトレース設定

## ベストプラクティス

1. **サンプリングルールの調整**: 本番環境ではサンプリング率を下げてコスト削減
2. **アノテーションの活用**: 検索に使用するフィールドはアノテーションで記録
3. **サブセグメントの作成**: 外部呼び出しやDB接続を詳細に記録
4. **グループの活用**: 重要なトレースをグループ化してモニタリング
5. **エラーハンドリング**: 例外情報をトレースに含める
6. **CloudWatch連携**: X-RayメトリクスをCloudWatchで監視

## X-Rayデーモン

### 役割

- アプリケーションからセグメントを受信
- バッファリングしてX-Ray APIに送信
- UDPポート2000でリッスン

### 実行環境別の設定

| 環境 | 設定方法 |
|------|----------|
| EC2 | デーモンをインストール・起動 |
| ECS/Fargate | サイドカーコンテナとして実行 |
| Lambda | レイヤーとして自動設定 |
| Elastic Beanstalk | 設定オプションで有効化 |

## サンプリングルール

### デフォルトルール

- **リザーバー**: 1秒あたり1リクエストを必ずサンプル
- **レート**: リザーバー後、5%をサンプル

### カスタムルール例

```json
{
  "SamplingRule": {
    "RuleName": "HighPriorityEndpoint",
    "Priority": 1,
    "ReservoirSize": 10,
    "FixedRate": 0.1,
    "URLPath": "/api/critical/*",
    "HTTPMethod": "*",
    "ServiceName": "*",
    "ServiceType": "*",
    "Host": "*"
  }
}
```

## アノテーション vs メタデータ

| 項目 | アノテーション | メタデータ |
|------|----------------|------------|
| インデックス | ✓（検索可能） | ❌ |
| 型 | 文字列、数値、Boolean | 任意のオブジェクト |
| 用途 | フィルタリング、検索 | 追加情報の保存 |
| 例 | user_id, environment | リクエストボディ、レスポンス |

## SDK使用例（Python）

```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# 自動パッチ（boto3, requests, sqlalchemy等）
patch_all()

# カスタムセグメント
@xray_recorder.capture('my_function')
def my_function():
    # サブセグメント
    with xray_recorder.in_subsegment('external_call') as subsegment:
        # アノテーション（検索用）
        subsegment.put_annotation('user_id', user_id)

        # メタデータ（追加情報）
        subsegment.put_metadata('request', request_data)

        # 処理
        result = external_api_call()

    return result
```

## サービス統合

### Lambda

```python
# Lambda関数でのX-Ray有効化
# 1. Lambdaコンソールでアクティブトレーシングを有効化
# 2. または SAM/CloudFormation で設定

# template.yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Tracing: Active
```

### API Gateway

```yaml
# API GatewayでX-Rayを有効化
MyApi:
  Type: AWS::Serverless::Api
  Properties:
    TracingEnabled: true
```

### ECS

```json
{
  "name": "xray-daemon",
  "image": "amazon/aws-xray-daemon",
  "portMappings": [
    {
      "containerPort": 2000,
      "protocol": "udp"
    }
  ]
}
```

## トレースヘッダー

```
X-Amzn-Trace-Id: Root=1-12345678-abcdef012345678901234567;Parent=1234567890abcdef;Sampled=1
```

| フィールド | 説明 |
|------------|------|
| Root | トレースID |
| Parent | 親セグメントID |
| Sampled | サンプリング決定（1=記録、0=記録しない） |

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| Lambda | 自動トレース（アクティブトレーシング） |
| API Gateway | 受信リクエストのトレース |
| ECS/Fargate | サイドカーデーモン |
| Elastic Beanstalk | プラットフォーム統合 |
| App Mesh | サービスメッシュトレース |
| CloudWatch | X-Rayメトリクス、ServiceLens |
| CloudWatch Logs | トレースとログの相関 |
| SNS/SQS | 分散メッセージングのトレース |
| DynamoDB/RDS | データベース呼び出しのトレース |

## ServiceLens

CloudWatchとX-Rayを統合したビュー:
- サービスマップ
- ログとトレースの相関
- メトリクス、ログ、トレースの統合ダッシュボード

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS X-Ray 開発者ガイド](https://docs.aws.amazon.com/xray/latest/devguide/)
- [X-Ray SDK for Python](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python.html)
- [サンプリングルール](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-sampling.html)
