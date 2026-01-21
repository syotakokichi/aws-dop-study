# Amazon Route 53

## 概要

Amazon Route 53は、高可用性と拡張性を備えたドメインネームシステム（DNS）Webサービス。ドメイン登録、DNSルーティング、ヘルスチェックの3つの主要機能を提供し、グローバルなトラフィック管理と高可用性アーキテクチャを実現する。

## キーコンセプト

- **ホストゾーン（Hosted Zone）**: ドメインのDNSレコードを管理するコンテナ
- **レコードセット**: DNS応答を定義（A, AAAA, CNAME, Alias等）
- **ルーティングポリシー**: トラフィックのルーティング方法
- **ヘルスチェック**: エンドポイントの健全性を監視
- **エイリアスレコード**: AWSリソースへの特別なレコードタイプ
- **トラフィックフロー**: 複雑なルーティングを視覚的に設計
- **リゾルバー**: VPC内のDNSクエリを処理

## 試験頻出ポイント

- [ ] ルーティングポリシーの種類と選択基準
- [ ] フェイルオーバールーティングの設定
- [ ] ヘルスチェックの種類と設定
- [ ] エイリアスレコード vs CNAMEレコード
- [ ] プライベートホストゾーンとVPC関連付け
- [ ] Route 53 Resolver（インバウンド/アウトバウンド）
- [ ] 地理的近接性ルーティング（Traffic Flow）
- [ ] DNSSECの設定

## DOP-C02での出題パターン

### パターン1: アクティブ-パッシブフェイルオーバー
- **シナリオ**: プライマリリージョン障害時にDRリージョンに切り替え
- **ポイント**: フェイルオーバールーティング + ヘルスチェック

### パターン2: レイテンシ最適化
- **シナリオ**: ユーザーに最も近いリージョンにルーティング
- **ポイント**: レイテンシベースルーティング

### パターン3: 段階的移行
- **シナリオ**: 新環境に10%のトラフィックを送って検証
- **ポイント**: 加重ルーティング

### パターン4: コンプライアンス対応
- **シナリオ**: 特定の国のユーザーを特定リージョンにルーティング
- **ポイント**: 地理的位置ルーティング

### パターン5: ハイブリッドDNS
- **シナリオ**: オンプレミスとAWS間でDNSクエリを解決
- **ポイント**: Route 53 Resolver + エンドポイント

## ベストプラクティス

1. **ヘルスチェックの活用**: 常にヘルスチェックと組み合わせてルーティング
2. **エイリアスレコードの使用**: AWSリソースにはエイリアスを優先
3. **フェイルオーバーの設定**: 重要なサービスには必ずフェイルオーバーを設定
4. **低TTLの使用**: フェイルオーバー時は低いTTLで素早く切り替え
5. **ヘルスチェックの分散**: 複数リージョンからチェック
6. **DNSSECの有効化**: DNSスプーフィング対策

## ルーティングポリシー

| ポリシー | 説明 | ユースケース |
|----------|------|--------------|
| シンプル | 単一リソースへルーティング | 単一エンドポイント |
| 加重 | 重みに応じて分配 | A/Bテスト、段階的移行 |
| レイテンシ | 最も低レイテンシのリージョンへ | グローバル分散 |
| フェイルオーバー | プライマリ障害時にセカンダリへ | DR |
| 地理的位置 | ユーザーの場所に基づく | コンプライアンス、ローカライゼーション |
| 地理的近接性 | 地理的距離+バイアスで制御 | 細かいトラフィック制御 |
| マルチバリュー | 複数の健全なレコードを返す | シンプルな負荷分散 |
| IPベース | クライアントIPに基づく | ISP/企業IPの区別 |

## フェイルオーバールーティング

### 設定例

```yaml
# プライマリレコード
PrimaryRecord:
  Type: AWS::Route53::RecordSet
  Properties:
    HostedZoneId: !Ref HostedZone
    Name: www.example.com
    Type: A
    SetIdentifier: primary
    Failover: PRIMARY
    HealthCheckId: !Ref PrimaryHealthCheck
    AliasTarget:
      HostedZoneId: !GetAtt PrimaryALB.CanonicalHostedZoneID
      DNSName: !GetAtt PrimaryALB.DNSName

# セカンダリレコード
SecondaryRecord:
  Type: AWS::Route53::RecordSet
  Properties:
    HostedZoneId: !Ref HostedZone
    Name: www.example.com
    Type: A
    SetIdentifier: secondary
    Failover: SECONDARY
    AliasTarget:
      HostedZoneId: !GetAtt SecondaryALB.CanonicalHostedZoneID
      DNSName: !GetAtt SecondaryALB.DNSName
```

## ヘルスチェック

### タイプ

| タイプ | 説明 |
|--------|------|
| エンドポイント | HTTP/HTTPS/TCPでエンドポイントを監視 |
| 計算済み | 他のヘルスチェックの状態を集約 |
| CloudWatch Alarm | CloudWatchアラームの状態を監視 |

### 設定例

```yaml
HealthCheck:
  Type: AWS::Route53::HealthCheck
  Properties:
    HealthCheckConfig:
      Type: HTTPS
      FullyQualifiedDomainName: www.example.com
      Port: 443
      ResourcePath: /health
      RequestInterval: 30
      FailureThreshold: 3
      EnableSNI: true
      Regions:
        - us-east-1
        - us-west-2
        - eu-west-1
```

### 計算済みヘルスチェック

```yaml
CalculatedHealthCheck:
  Type: AWS::Route53::HealthCheck
  Properties:
    HealthCheckConfig:
      Type: CALCULATED
      HealthThreshold: 2  # 最低2つが健全なら健全
      ChildHealthChecks:
        - !Ref HealthCheck1
        - !Ref HealthCheck2
        - !Ref HealthCheck3
```

## エイリアスレコード vs CNAME

| 項目 | エイリアス | CNAME |
|------|------------|-------|
| Zone Apex | ✓ | ❌ |
| AWSリソース統合 | 最適化 | 通常 |
| クエリ課金 | 無料 | 有料 |
| TTL | ターゲットに従う | 設定可能 |
| 対象 | AWSリソースのみ | 任意のドメイン |

### エイリアス対応リソース

- ELB (ALB, NLB, CLB)
- CloudFront
- API Gateway
- S3 (静的ウェブサイトホスティング)
- Elastic Beanstalk
- VPC Interface Endpoints
- Global Accelerator
- 同一ホストゾーンのRoute 53レコード

## Route 53 Resolver

### コンポーネント

| コンポーネント | 説明 |
|----------------|------|
| インバウンドエンドポイント | オンプレミス → AWS VPCへのDNSクエリ |
| アウトバウンドエンドポイント | AWS VPC → オンプレミスへのDNSクエリ |
| リゾルバールール | 条件付きフォワーディング |

### ハイブリッドDNSアーキテクチャ

```
オンプレミス                        AWS VPC
┌─────────────┐                ┌─────────────────────┐
│  DNS Server │←── Inbound ←──│ Route 53 Resolver   │
│             │                │                     │
│             │── Outbound ───→│ (aws.example.com)   │
└─────────────┘                └─────────────────────┘
(corp.example.com)
```

### 設定例

```yaml
# アウトバウンドエンドポイント
OutboundEndpoint:
  Type: AWS::Route53Resolver::ResolverEndpoint
  Properties:
    Direction: OUTBOUND
    IpAddresses:
      - SubnetId: !Ref PrivateSubnet1
      - SubnetId: !Ref PrivateSubnet2
    SecurityGroupIds:
      - !Ref ResolverSG

# フォワーディングルール
ForwardingRule:
  Type: AWS::Route53Resolver::ResolverRule
  Properties:
    DomainName: corp.example.com
    RuleType: FORWARD
    ResolverEndpointId: !Ref OutboundEndpoint
    TargetIps:
      - Ip: 10.0.0.10
        Port: 53
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| CloudFront | エイリアスレコード |
| ELB | エイリアスレコード |
| S3 | 静的ウェブサイトホスティング |
| API Gateway | カスタムドメイン |
| CloudWatch | ヘルスチェックメトリクス |
| SNS | ヘルスチェックアラーム通知 |
| ACM | SSL証明書検証 |
| VPC | プライベートホストゾーン |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [Amazon Route 53 開発者ガイド](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/)
- [ルーティングポリシー](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- [ヘルスチェック](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html)
- [Route 53 Resolver](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html)
