# ドメイン1: SDLC自動化 (22%)

## 概要

ソフトウェア開発ライフサイクル（SDLC）の自動化に関するドメイン。
CI/CDパイプラインの設計・実装・最適化が中心。

## 主要サービス

| サービス | 役割 |
|----------|------|
| CodeCommit | ソースコード管理（Git） |
| CodeBuild | ビルド・テスト自動化 |
| CodeDeploy | デプロイ自動化 |
| CodePipeline | CI/CDオーケストレーション |
| CodeArtifact | アーティファクト管理 |

## 試験頻出トピック

### デプロイ戦略
- **Blue/Green**: ダウンタイムゼロ、即時ロールバック可能
- **Canary**: 段階的リリース、リスク軽減
- **Rolling**: 順次更新、リソース効率的
- **In-Place**: 既存インスタンス上で更新

### CodePipeline
- [ ] ステージとアクション
- [ ] 承認アクション
- [ ] クロスアカウントパイプライン
- [ ] EventBridgeトリガー
- [ ] アーティファクト管理

### CodeBuild
- [ ] buildspec.yml
- [ ] 環境変数とSecrets Manager連携
- [ ] VPC内ビルド
- [ ] カスタムイメージ
- [ ] キャッシュ戦略

### CodeDeploy
- [ ] AppSpec.yml
- [ ] ライフサイクルフック
- [ ] デプロイグループ
- [ ] EC2/オンプレミス vs ECS vs Lambda

## SRE本との関連

- **第8章: リリースエンジニアリング** - CI/CDの設計原則
- **第3章: リスクの受容** - カナリアリリースとエラーバジェット

## 学習ファイル

- [codepipeline.md](./codepipeline.md)
- [codebuild.md](./codebuild.md)
- [codecommit.md](./codecommit.md)
- [codedeploy.md](./codedeploy.md)
- [codeartifact.md](./codeartifact.md)

## ハンズオン

- [hands-on/](./hands-on/) - 実践記録
