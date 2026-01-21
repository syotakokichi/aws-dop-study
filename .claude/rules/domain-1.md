---
globs:
  - "domain-1-sdlc-automation/**"
  - "**/codepipeline*"
  - "**/codebuild*"
  - "**/codecommit*"
  - "**/codedeploy*"
  - "**/codeartifact*"
---

# ドメイン1: SDLC自動化 ルール

## 配点: 22%（最重要ドメイン）

このドメインの質問に回答する際は、以下のファイルを優先的に参照:

- domain-1-sdlc-automation/codepipeline.md
- domain-1-sdlc-automation/codebuild.md
- domain-1-sdlc-automation/codecommit.md
- domain-1-sdlc-automation/codedeploy.md
- domain-1-sdlc-automation/codeartifact.md

## 頻出トピック

1. **CI/CDパイプライン設計**
   - CodePipeline のステージ構成
   - クロスアカウント/クロスリージョンデプロイ
   - 承認ワークフロー

2. **デプロイ戦略**
   - Blue/Green vs Canary vs Rolling
   - CodeDeploy の AppSpec.yml
   - ロールバック設定

3. **ビルド自動化**
   - buildspec.yml の構造
   - キャッシュ戦略
   - シークレット管理

## 回答時の注意

- CodeCommit は新規作成終了のため、移行先（GitHub等）も言及
- デプロイ戦略の選択理由を必ず説明
- クロスアカウント設定は IAM ロールと KMS の両方に言及
