# AWS CodeArtifact

## 概要

AWS CodeArtifactは、ソフトウェアパッケージを安全に保存、公開、共有するためのフルマネージドのアーティファクトリポジトリサービス。npm、PyPI、Maven、NuGet等の一般的なパッケージマネージャーと互換性があり、組織全体でパッケージを一元管理できる。

## キーコンセプト

- **ドメイン（Domain）**: リポジトリをグループ化する最上位の組織単位
- **リポジトリ（Repository）**: パッケージを格納するストレージ
- **アップストリームリポジトリ**: 他のリポジトリからパッケージを取得するための参照先
- **外部接続（External Connection）**: 公開リポジトリ（npm, PyPI等）への接続
- **パッケージ**: ソフトウェアコンポーネント（依存関係を含むバージョン管理されたアーティファクト）
- **アセット**: パッケージに含まれる個々のファイル
- **パッケージグループ**: パッケージのアクセス制御をまとめて管理

## 試験頻出ポイント

- [ ] ドメインとリポジトリの階層構造
- [ ] アップストリームリポジトリの設定と動作
- [ ] 外部接続を使用した公開リポジトリとの連携
- [ ] パッケージのキャッシュ動作
- [ ] クロスアカウントアクセスの設定
- [ ] 認証トークンの取得と使用
- [ ] パッケージの公開制御（オリジンコントロール）
- [ ] VPCエンドポイントを使用したプライベートアクセス

## DOP-C02での出題パターン

### パターン1: プライベートパッケージと公開パッケージの統合
- **シナリオ**: 社内パッケージとnpmjsの両方を単一のリポジトリで管理したい
- **ポイント**: 外部接続 + アップストリームリポジトリの設定

### パターン2: クロスアカウントパッケージ共有
- **シナリオ**: 複数のAWSアカウント間で共通ライブラリを共有
- **ポイント**: ドメインポリシーとリポジトリポリシーの設定

### パターン3: CI/CDパイプラインとの統合
- **シナリオ**: CodeBuildでパッケージをビルドしてCodeArtifactに公開
- **ポイント**: 認証トークンの取得、IAMロールの設定

### パターン4: セキュリティ要件への対応
- **シナリオ**: インターネットアクセスなしでパッケージを取得
- **ポイント**: VPCエンドポイント、パッケージのキャッシュ

## ベストプラクティス

1. **ドメインでの一元管理**: 組織全体で1つのドメインを使用してポリシーを統一
2. **アップストリームの活用**: 内部リポジトリ → 外部接続リポジトリの順でチェーン
3. **オリジンコントロール**: 外部パッケージの意図しない上書きを防止
4. **VPCエンドポイント**: セキュアな環境ではプライベートアクセスを使用
5. **認証トークンの短期間利用**: トークンは最大12時間有効、定期的に更新
6. **パッケージグループでアクセス制御**: まとめて権限管理

## アーキテクチャパターン

### 推奨構成

```
[開発者/CI/CD]
      ↓
[内部リポジトリ（my-repo）]
      ↓ (アップストリーム)
[外部接続リポジトリ（npm-store）]
      ↓ (外部接続)
[public npm registry]
```

### 動作フロー

1. パッケージ要求が内部リポジトリに到着
2. 内部リポジトリにパッケージがあれば返却
3. なければアップストリームリポジトリをチェック
4. 最終的に外部接続から公開リポジトリに問い合わせ
5. 取得したパッケージはキャッシュされる

## サポートするパッケージ形式

| パッケージマネージャー | 形式 | 外部接続先 |
|------------------------|------|------------|
| npm | JavaScript/TypeScript | npmjs.com |
| pip/twine | Python | PyPI |
| Maven/Gradle | Java | Maven Central |
| NuGet | .NET | NuGet Gallery |
| Swift | Swift | - |
| Ruby (Gem) | Ruby | RubyGems |
| Cargo | Rust | crates.io |

## 認証とアクセス

### 認証トークンの取得（CLI）

```bash
# npm用
aws codeartifact login --tool npm \
  --domain my-domain \
  --domain-owner 123456789012 \
  --repository my-repo

# pip用
aws codeartifact login --tool pip \
  --domain my-domain \
  --domain-owner 123456789012 \
  --repository my-repo

# トークンのみ取得
aws codeartifact get-authorization-token \
  --domain my-domain \
  --domain-owner 123456789012 \
  --query authorizationToken \
  --output text
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| CodeBuild | パッケージのビルドと公開 |
| CodePipeline | CI/CDパイプラインでの利用 |
| IAM | 認証とアクセス制御 |
| KMS | パッケージの暗号化 |
| CloudWatch | メトリクスとログ |
| CloudTrail | API監査ログ |
| VPC Endpoints | プライベートアクセス |
| Organizations | クロスアカウント管理 |

## ポリシー例

### ドメインポリシー（クロスアカウント）

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:root"
      },
      "Action": [
        "codeartifact:GetDomainPermissionsPolicy",
        "codeartifact:ListRepositoriesInDomain",
        "codeartifact:GetAuthorizationToken",
        "codeartifact:DescribeDomain",
        "codeartifact:CreateRepository"
      ],
      "Resource": "*"
    }
  ]
}
```

## オリジンコントロール

| 設定 | 説明 |
|------|------|
| ALLOW | 外部からの公開を許可 |
| BLOCK | 外部からの公開をブロック（キャッシュのみ） |

**ユースケース**: 外部接続リポジトリでは通常BLOCKを設定し、パッケージの意図しない上書きを防止

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS CodeArtifact ユーザーガイド](https://docs.aws.amazon.com/codeartifact/latest/ug/)
- [CodeArtifact と npm の使用](https://docs.aws.amazon.com/codeartifact/latest/ug/using-npm.html)
- [アップストリームリポジトリ](https://docs.aws.amazon.com/codeartifact/latest/ug/repos-upstream.html)
