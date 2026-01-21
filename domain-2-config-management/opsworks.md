# AWS OpsWorks

## 概要

AWS OpsWorksは、ChefまたはPuppetを使用してインフラストラクチャの構成管理を行うサービス。3つのバリエーションがあり、既存のChef/Puppetの知識を活かした構成管理が可能。

**重要**: OpsWorks Stacksは2024年5月26日に新規スタック作成を終了。試験では移行先としてSystems ManagerやChef/Puppet自前運用が問われることがある。

## バリエーション

| サービス | 説明 | 状態 |
|----------|------|------|
| OpsWorks for Chef Automate | フルマネージドChefサーバー | 提供中 |
| OpsWorks for Puppet Enterprise | フルマネージドPuppetサーバー | 提供中 |
| OpsWorks Stacks | AWS独自のレイヤー型管理 | 新規作成終了 |

## キーコンセプト

### OpsWorks Stacks（レガシー）

- **スタック**: アプリケーション全体を表す最上位の概念
- **レイヤー**: 機能ごとのグループ（Web Server, App Server, DB等）
- **インスタンス**: レイヤーに属するEC2インスタンス
- **アプリケーション**: デプロイするアプリコード
- **Cookbooks/Recipes**: Chefの構成定義

### ライフサイクルイベント（OpsWorks Stacks）

| イベント | タイミング |
|----------|------------|
| Setup | インスタンス起動完了後 |
| Configure | スタック構成変更時（全インスタンスで実行） |
| Deploy | アプリケーションデプロイ時 |
| Undeploy | アプリケーション削除時 |
| Shutdown | インスタンス停止前 |

## 試験頻出ポイント

- [ ] OpsWorks Stacks、Chef Automate、Puppet Enterpriseの違い
- [ ] ライフサイクルイベントの種類とタイミング
- [ ] Auto Healingの仕組み
- [ ] Time-based / Load-based インスタンス
- [ ] Systems Managerへの移行パターン
- [ ] ChefレシピとPuppetマニフェストの基本

## DOP-C02での出題パターン

### パターン1: 構成管理ツールの選定
- **シナリオ**: 既存のChef環境をAWSに移行
- **ポイント**: OpsWorks for Chef Automate

### パターン2: OpsWorks Stacksからの移行
- **シナリオ**: OpsWorks Stacksの廃止に対応
- **ポイント**: Systems Manager + Automation/State Manager

### パターン3: Auto Healing
- **シナリオ**: インスタンス障害時に自動復旧
- **ポイント**: OpsWorksのAuto Healing機能

### パターン4: スケーリング戦略
- **シナリオ**: 時間帯やロードに応じたインスタンス数調整
- **ポイント**: Time-based / Load-based インスタンス

## OpsWorks for Chef Automate

### 特徴

- フルマネージドChefサーバー
- Chef Infra、Chef InSpec、Chef Habitatをサポート
- 自動バックアップ
- システムメンテナンスの自動化

### 構成要素

- **Chef Server**: 構成情報を管理する中央サーバー
- **Chef Workstation**: Cookbookの開発環境
- **Chef Nodes**: 構成を適用するサーバー（EC2等）
- **Cookbooks**: 構成定義（Rubyベース）
- **Knife**: Chef CLIツール

## OpsWorks for Puppet Enterprise

### 特徴

- フルマネージドPuppetサーバー
- Puppet Enterprise機能をフルサポート
- 自動バックアップ
- オーケストレーション機能

### 構成要素

- **Puppet Master**: 構成情報を管理する中央サーバー
- **Puppet Agents**: 構成を適用するノード
- **Modules**: 構成定義
- **Manifests**: Puppetの設定ファイル
- **Facter**: ノード情報を収集

## OpsWorks Stacks（レガシー）

### インスタンスタイプ

| タイプ | 説明 |
|--------|------|
| 24/7 | 常時稼働 |
| Time-based | スケジュールで起動/停止 |
| Load-based | CPU/メモリ使用率でスケール |

### Auto Healing

- インスタンスのヘルスチェック
- 障害検知時に自動再起動
- EBSボリュームの再アタッチ
- 設定したレシピの再実行

## 移行先の選択

### OpsWorks Stacksからの移行

| 用途 | 移行先 |
|------|--------|
| 一般的な構成管理 | Systems Manager State Manager |
| コマンド実行 | Systems Manager Run Command |
| 自動化ワークフロー | Systems Manager Automation |
| パッチ管理 | Systems Manager Patch Manager |
| Chefを継続使用 | OpsWorks for Chef Automate / 自前運用 |
| Puppetを継続使用 | OpsWorks for Puppet Enterprise / 自前運用 |

## Chef vs Puppet vs Systems Manager

| 項目 | Chef | Puppet | Systems Manager |
|------|------|--------|-----------------|
| 言語 | Ruby (DSL) | Puppet DSL | YAML/JSON |
| アーキテクチャ | Pull型 | Pull型 | Push/Pull |
| エージェント | chef-client | puppet-agent | SSM Agent |
| AWS統合 | OpsWorks | OpsWorks | ネイティブ |
| 学習コスト | 中〜高 | 中〜高 | 低 |

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| EC2 | マネージドインスタンス |
| ELB | ロードバランサー統合 |
| RDS | データベースレイヤー |
| CloudWatch | メトリクス/ログ |
| IAM | アクセス制御 |
| S3 | Cookbook保存 |
| Systems Manager | 移行先としての連携 |

## ベストプラクティス

1. **新規プロジェクトでは避ける**: OpsWorks Stacksは新規作成不可
2. **Chef/Puppet継続時**: OpsWorks for Chef Automate/Puppet Enterpriseを検討
3. **シンプルな構成管理**: Systems Managerへの移行を推奨
4. **既存環境**: 段階的な移行計画を策定

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS OpsWorks ユーザーガイド](https://docs.aws.amazon.com/opsworks/latest/userguide/)
- [OpsWorks for Chef Automate](https://docs.aws.amazon.com/opsworks/latest/userguide/welcome_opscm.html)
- [OpsWorks for Puppet Enterprise](https://docs.aws.amazon.com/opsworks/latest/userguide/welcome_opspup.html)
- [Chef ドキュメント](https://docs.chef.io/)
- [Puppet ドキュメント](https://puppet.com/docs/)
