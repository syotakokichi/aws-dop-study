# Elastic Load Balancing (ELB)

## 概要

Elastic Load Balancingは、受信トラフィックを複数のターゲット（EC2、コンテナ、Lambda等）に自動的に分散するマネージドロードバランサーサービス。高可用性、自動スケーリング、セキュリティ機能を提供し、アプリケーションの耐障害性を向上させる。

## ロードバランサータイプ

| タイプ | レイヤー | プロトコル | ユースケース |
|--------|---------|------------|--------------|
| ALB (Application) | L7 | HTTP/HTTPS/gRPC | Webアプリ、マイクロサービス |
| NLB (Network) | L4 | TCP/UDP/TLS | 低レイテンシ、高スループット |
| GLB (Gateway) | L3 | IP | サードパーティアプライアンス |
| CLB (Classic) | L4/L7 | - | レガシー（非推奨） |

## キーコンセプト

- **リスナー**: 接続リクエストをチェックするポート/プロトコル設定
- **ターゲットグループ**: トラフィックの宛先（インスタンス、IP、Lambda）
- **ヘルスチェック**: ターゲットの健全性を確認
- **ルール**: ALBでのリクエストルーティング条件
- **スティッキーセッション**: 同一クライアントを同一ターゲットに送信
- **クロスゾーン負荷分散**: AZ間で均等にトラフィックを分散
- **接続ドレイン**: 登録解除時に既存接続を完了させる

## 試験頻出ポイント

- [ ] ALB vs NLB vs GLBの選択基準
- [ ] ALBのパスベース/ホストベースルーティング
- [ ] NLBの静的IP/PrivateLink対応
- [ ] ヘルスチェックの設定と閾値
- [ ] クロスゾーン負荷分散の動作
- [ ] SSL/TLS終端の設定
- [ ] WAF統合（ALBのみ）
- [ ] ターゲットタイプ（instance, ip, lambda）
- [ ] 接続ドレイン（Deregistration Delay）

## DOP-C02での出題パターン

### パターン1: マイクロサービスのルーティング
- **シナリオ**: パスに応じて異なるサービスにルーティング
- **ポイント**: ALB + パスベースルーティング

### パターン2: 低レイテンシ要件
- **シナリオ**: ミリ秒単位のレイテンシが要求される
- **ポイント**: NLB（TCP/UDP直接処理）

### パターン3: 静的IP要件
- **シナリオ**: ファイアウォールのホワイトリストに固定IPが必要
- **ポイント**: NLB + Elastic IP

### パターン4: サードパーティアプライアンス
- **シナリオ**: ファイアウォール/IDSをトラフィックに挿入
- **ポイント**: GLB + GENEVE

### パターン5: Blue/Greenデプロイ
- **シナリオ**: 加重ルーティングで段階的に新バージョンへ移行
- **ポイント**: ALB + 加重ターゲットグループ

## ベストプラクティス

1. **複数AZへのデプロイ**: 耐障害性向上
2. **適切なヘルスチェック設定**: アプリケーションの実際の健全性を確認
3. **クロスゾーン負荷分散の有効化**: 均等な負荷分散
4. **接続ドレインの設定**: グレースフルシャットダウン
5. **WAFの統合**: Webアプリケーション保護（ALB）
6. **アクセスログの有効化**: トラブルシューティングと監査

## Application Load Balancer (ALB)

### 特徴

- **L7ルーティング**: パス、ホスト、HTTPヘッダー、クエリ文字列等
- **WebSocket/HTTP/2対応**
- **Lambda統合**: サーバーレスバックエンド
- **WAF統合**: Webアプリケーションファイアウォール
- **認証統合**: Cognito、OIDC

### ルーティング条件

| 条件タイプ | 説明 |
|------------|------|
| host-header | ホスト名 (api.example.com) |
| path-pattern | パス (/api/*, /images/*) |
| http-header | カスタムヘッダー |
| http-request-method | GET, POST等 |
| query-string | クエリパラメータ |
| source-ip | 送信元IP |

### リスナールール例

```yaml
# CloudFormation
ListenerRule:
  Type: AWS::ElasticLoadBalancingV2::ListenerRule
  Properties:
    ListenerArn: !Ref HttpsListener
    Priority: 1
    Conditions:
      - Field: path-pattern
        Values:
          - /api/*
    Actions:
      - Type: forward
        TargetGroupArn: !Ref ApiTargetGroup
```

### 加重ターゲットグループ（Blue/Green）

```yaml
Actions:
  - Type: forward
    ForwardConfig:
      TargetGroups:
        - TargetGroupArn: !Ref BlueTargetGroup
          Weight: 90
        - TargetGroupArn: !Ref GreenTargetGroup
          Weight: 10
```

## Network Load Balancer (NLB)

### 特徴

- **超低レイテンシ**: 数百万リクエスト/秒
- **静的IP**: AZごとに1つのElastic IP
- **TCP/UDP**: L4直接処理
- **PrivateLink**: VPCエンドポイントサービスのフロントエンド
- **クライアントIP保持**: X-Forwarded-For不要

### ユースケース

- 金融取引システム
- ゲームサーバー
- IoTデバイス接続
- VPCエンドポイントサービス

### 設定例

```yaml
NetworkLoadBalancer:
  Type: AWS::ElasticLoadBalancingV2::LoadBalancer
  Properties:
    Type: network
    Scheme: internet-facing
    Subnets:
      - !Ref PublicSubnet1
      - !Ref PublicSubnet2
    LoadBalancerAttributes:
      - Key: load_balancing.cross_zone.enabled
        Value: "true"
```

## Gateway Load Balancer (GLB)

### 特徴

- **L3処理**: IPパケットレベル
- **GENEVE**: カプセル化プロトコル
- **透過的挿入**: インライン検査
- **スケーラブル**: サードパーティアプライアンスの負荷分散

### アーキテクチャ

```
インターネット
    ↓
IGW → GLB エンドポイント → GLB → アプライアンス（ファイアウォール等）
                                        ↓
                                    実際のターゲット
```

## ヘルスチェック

### 設定項目

| 項目 | 説明 | ALB | NLB |
|------|------|-----|-----|
| Protocol | チェックプロトコル | HTTP/HTTPS | TCP/HTTP/HTTPS |
| Path | チェックパス | /health | - |
| Port | チェックポート | traffic-port | traffic-port |
| HealthyThreshold | 健全とみなす連続成功数 | 2-10 | 2-10 |
| UnhealthyThreshold | 異常とみなす連続失敗数 | 2-10 | 2-10 |
| Timeout | タイムアウト | 2-120秒 | 6秒（固定） |
| Interval | チェック間隔 | 5-300秒 | 10/30秒 |

### 設定例

```yaml
TargetGroup:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    HealthCheckEnabled: true
    HealthCheckPath: /health
    HealthCheckProtocol: HTTP
    HealthCheckPort: traffic-port
    HealthyThresholdCount: 2
    UnhealthyThresholdCount: 3
    HealthCheckIntervalSeconds: 30
    HealthCheckTimeoutSeconds: 5
    Matcher:
      HttpCode: '200-299'
```

## SSL/TLS

### 設定

- **SSL証明書**: ACM（推奨）またはIAM
- **セキュリティポリシー**: TLSバージョン、暗号スイート
- **SNI**: 複数証明書のサポート

### 終端パターン

| パターン | 説明 |
|----------|------|
| ELBで終端 | ELB → ターゲット（HTTP） |
| パススルー | NLBでTCPパススルー |
| 再暗号化 | ELB → ターゲット（HTTPS） |

## アクセスログ

```yaml
LoadBalancerAttributes:
  - Key: access_logs.s3.enabled
    Value: "true"
  - Key: access_logs.s3.bucket
    Value: my-logs-bucket
  - Key: access_logs.s3.prefix
    Value: alb-logs
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| Auto Scaling | ターゲットグループへの自動登録 |
| ECS/Fargate | サービスディスカバリ |
| Lambda | ALBターゲット |
| WAF | Webアプリケーション保護（ALB） |
| Shield | DDoS保護 |
| ACM | SSL証明書 |
| Route 53 | DNSルーティング |
| CloudWatch | メトリクス/アラーム |
| VPC | セキュリティグループ、サブネット |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [Elastic Load Balancing ユーザーガイド](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/)
- [ALB ユーザーガイド](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [NLB ユーザーガイド](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/)
- [GLB ユーザーガイド](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/)
