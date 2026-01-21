# AWS CodeBuild

## 概要

AWS CodeBuildは、ソースコードのコンパイル、テスト実行、デプロイ可能なアーティファクトの生成を行うフルマネージドのビルドサービス。サーバーのプロビジョニングや管理が不要で、需要に応じて自動スケールする。

## キーコンセプト

- **ビルドプロジェクト**: ビルドの設定を定義（ソース、環境、buildspec等）
- **buildspec.yml**: ビルドコマンドと設定を定義するファイル
- **ビルド環境**: ビルドを実行するDockerコンテナ（マネージドイメージまたはカスタムイメージ）
- **コンピューティングタイプ**: ビルドに割り当てるCPU/メモリ（small, medium, large, 2xlarge）
- **環境変数**: ビルド時に使用する変数（プレーンテキスト、Parameter Store、Secrets Manager）
- **ビルドフェーズ**: install, pre_build, build, post_build
- **キャッシュ**: ビルド時間短縮のための依存関係キャッシュ（S3またはローカル）
- **バッチビルド**: 複数のビルドを同時実行
- **VPC内ビルド**: プライベートリソースにアクセスするためのVPC設定

## 試験頻出ポイント

- [ ] buildspec.ymlの構造とフェーズ（phases, artifacts, cache）
- [ ] 環境変数の種類とセキュリティ（Parameter Store、Secrets Manager連携）
- [ ] ビルドキャッシュの設定（S3キャッシュ、ローカルキャッシュ）
- [ ] VPC内でのビルド設定（プライベートリソースアクセス）
- [ ] カスタムビルドイメージの使用
- [ ] ビルドバッジの設定
- [ ] レポート機能（テストレポート、コードカバレッジ）
- [ ] ビルドタイムアウトの設定

## DOP-C02での出題パターン

### パターン1: シークレットの安全な管理
- **シナリオ**: ビルド中にデータベースパスワードが必要
- **ポイント**: Secrets Manager/Parameter Storeから環境変数として取得

### パターン2: ビルド時間の最適化
- **シナリオ**: ビルドが遅く、依存関係のダウンロードに時間がかかる
- **ポイント**: S3キャッシュまたはローカルキャッシュの活用

### パターン3: プライベートリソースへのアクセス
- **シナリオ**: ビルド中にVPC内のRDSにアクセスが必要
- **ポイント**: VPC設定、セキュリティグループ、NATゲートウェイ

### パターン4: マルチプラットフォームビルド
- **シナリオ**: 同一ソースからLinux/Windows両方のアーティファクトを生成
- **ポイント**: バッチビルドの活用

## ベストプラクティス

1. **buildspec.ymlをソースに含める**: バージョン管理されたビルド定義
2. **シークレットは環境変数で参照**: Parameter Store/Secrets Managerを使用
3. **キャッシュを活用**: 依存関係をキャッシュしてビルド時間短縮
4. **最小限のIAM権限**: ビルドに必要な権限のみ付与
5. **タイムアウトを適切に設定**: 無限ループを防止
6. **ログを保持**: CloudWatch Logsへのログ保存を有効化

## buildspec.yml構造

```yaml
version: 0.2

env:
  variables:
    KEY: "value"
  parameter-store:
    DB_PASSWORD: "/myapp/db/password"
  secrets-manager:
    API_KEY: "arn:aws:secretsmanager:..."

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install
  pre_build:
    commands:
      - npm run lint
  build:
    commands:
      - npm run build
  post_build:
    commands:
      - npm run test

artifacts:
  files:
    - '**/*'
  base-directory: dist

cache:
  paths:
    - node_modules/**/*

reports:
  test-reports:
    files:
      - '**/*'
    base-directory: test-results
    file-format: JUNITXML
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| CodePipeline | ビルドステージとして統合 |
| CodeCommit | ソースプロバイダー |
| S3 | ソース、アーティファクト、キャッシュストレージ |
| ECR | カスタムビルドイメージ、ビルド成果物 |
| Parameter Store | 設定値の安全な取得 |
| Secrets Manager | シークレットの安全な取得 |
| CloudWatch Logs | ビルドログの保存 |
| VPC | プライベートリソースへのアクセス |
| IAM | ビルドロールの権限管理 |

## コンピューティングタイプ

| タイプ | vCPU | メモリ | ユースケース |
|--------|------|--------|--------------|
| BUILD_GENERAL1_SMALL | 3 | 3 GB | 小規模ビルド |
| BUILD_GENERAL1_MEDIUM | 7 | 15 GB | 標準的なビルド |
| BUILD_GENERAL1_LARGE | 15 | 70 GB | 大規模ビルド |
| BUILD_GENERAL1_2XLARGE | 145 | 255 GB | 超大規模ビルド |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS CodeBuild ユーザーガイド](https://docs.aws.amazon.com/codebuild/latest/userguide/)
- [buildspec リファレンス](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)
- [CodeBuild のビルド環境](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref.html)
