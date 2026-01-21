# AWS CDK (Cloud Development Kit)

## 概要

AWS CDKは、プログラミング言語（TypeScript, Python, Java, C#, Go等）を使用してクラウドインフラストラクチャを定義できるオープンソースのフレームワーク。CloudFormationテンプレートを生成し、高レベルの抽象化（Constructs）により生産性を向上させる。

## キーコンセプト

- **App**: CDKアプリケーションのルート（複数のスタックを含む）
- **Stack**: CloudFormationスタックに対応するデプロイ単位
- **Construct**: 再利用可能なクラウドコンポーネント
- **L1 Constructs（Cfn）**: CloudFormationリソースの1:1マッピング
- **L2 Constructs**: ベストプラクティスを組み込んだ高レベル抽象化
- **L3 Constructs（Patterns）**: 複数リソースを含む設計パターン
- **Assets**: Lambda関数コード、Dockerイメージなどのローカルファイル
- **Context**: 環境固有の設定値
- **Aspects**: スタック全体に横断的な変更を適用

## 試験頻出ポイント

- [ ] cdk synth, cdk diff, cdk deployの役割
- [ ] Constructレベル（L1, L2, L3）の違いと使い分け
- [ ] cdk bootstrapの目的と仕組み
- [ ] CloudFormationとの関係（CDKはCFnを生成）
- [ ] Assetsの処理（S3へのアップロード）
- [ ] クロススタック参照の方法
- [ ] cdk.jsonとcontext設定
- [ ] テストの書き方（Assertions）

## DOP-C02での出題パターン

### パターン1: CDK vs CloudFormation
- **シナリオ**: IaCツールの選定
- **ポイント**: CDKはプログラミング言語の利点（ループ、条件、型安全性）を活かせる

### パターン2: 既存CloudFormationからの移行
- **シナリオ**: 既存のCloudFormationスタックをCDKで管理したい
- **ポイント**: cdk importコマンド、L1 Constructsの使用

### パターン3: CI/CDとの統合
- **シナリオ**: CDKをCodePipelineで自動デプロイ
- **ポイント**: CDK Pipelines（セルフミューテーション）

### パターン4: マルチ環境デプロイ
- **シナリオ**: 開発・ステージング・本番で同じインフラを展開
- **ポイント**: Contextによる環境切り替え、スタックのパラメータ化

## ベストプラクティス

1. **L2 Constructsを優先**: ベストプラクティスが組み込まれている
2. **環境ごとのスタック分離**: 開発/本番を別スタックで管理
3. **CDK Pipelinesの活用**: CI/CDをコードで定義
4. **Assetsの活用**: Lambda関数コードやDockerイメージを自動管理
5. **テストの実装**: Assertionsでインフラコードをテスト
6. **cdk diffの活用**: デプロイ前に変更内容を確認

## CDKコマンド

| コマンド | 説明 |
|----------|------|
| cdk init | 新規CDKプロジェクトを作成 |
| cdk synth | CloudFormationテンプレートを生成 |
| cdk diff | デプロイ済みスタックとの差分を表示 |
| cdk deploy | スタックをデプロイ |
| cdk destroy | スタックを削除 |
| cdk bootstrap | CDKツールキットスタックを作成 |
| cdk watch | ファイル変更を監視して自動デプロイ |
| cdk import | 既存リソースをスタックにインポート |

## Constructレベル

| レベル | 説明 | 例 |
|--------|------|-----|
| L1 (Cfn*) | CloudFormationリソースの1:1対応 | CfnBucket |
| L2 | デフォルト設定・便利メソッド付き | Bucket |
| L3 (Patterns) | 複数リソースの設計パターン | ApplicationLoadBalancedFargateService |

## コード例（TypeScript）

```typescript
import * as cdk from 'aws-cdk-lib';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import { Construct } from 'constructs';

export class MyStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // L2 Construct - ベストプラクティスが組み込まれている
    const bucket = new s3.Bucket(this, 'MyBucket', {
      versioned: true,
      encryption: s3.BucketEncryption.S3_MANAGED,
      removalPolicy: cdk.RemovalPolicy.RETAIN,
    });

    // Lambda関数（Assets自動処理）
    const fn = new lambda.Function(this, 'MyFunction', {
      runtime: lambda.Runtime.NODEJS_18_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda'), // ローカルディレクトリ
    });

    // 権限付与（L2の便利メソッド）
    bucket.grantRead(fn);

    // 出力
    new cdk.CfnOutput(this, 'BucketName', {
      value: bucket.bucketName,
      exportName: 'MyBucketName',
    });
  }
}
```

## CDK Pipelines

```typescript
import { CodePipeline, CodePipelineSource, ShellStep } from 'aws-cdk-lib/pipelines';

const pipeline = new CodePipeline(this, 'Pipeline', {
  pipelineName: 'MyPipeline',
  synth: new ShellStep('Synth', {
    input: CodePipelineSource.gitHub('owner/repo', 'main'),
    commands: [
      'npm ci',
      'npm run build',
      'npx cdk synth',
    ],
  }),
});

// ステージを追加
pipeline.addStage(new MyAppStage(this, 'Prod', {
  env: { account: '123456789012', region: 'ap-northeast-1' },
}));
```

## cdk.json設定

```json
{
  "app": "npx ts-node --prefer-ts-exts bin/my-app.ts",
  "context": {
    "@aws-cdk/core:stackRelativeExports": true,
    "environment": "dev",
    "vpcId": "vpc-12345678"
  }
}
```

## テスト（Assertions）

```typescript
import * as cdk from 'aws-cdk-lib';
import { Template } from 'aws-cdk-lib/assertions';
import { MyStack } from '../lib/my-stack';

test('S3 Bucket Created', () => {
  const app = new cdk.App();
  const stack = new MyStack(app, 'TestStack');
  const template = Template.fromStack(stack);

  // リソースの存在確認
  template.hasResourceProperties('AWS::S3::Bucket', {
    VersioningConfiguration: {
      Status: 'Enabled',
    },
  });

  // リソース数の確認
  template.resourceCountIs('AWS::S3::Bucket', 1);
});
```

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| CloudFormation | テンプレート生成先 |
| S3 | Assets（Lambda関数コード等）の保存 |
| ECR | Dockerイメージの保存 |
| CodePipeline | CDK Pipelinesでのデプロイ |
| CodeBuild | synthとdeploy処理 |
| IAM | Bootstrapで作成されるロール |
| SSM Parameter Store | Context値の取得 |

## Bootstrap

`cdk bootstrap`で作成されるリソース:
- **S3バケット**: Assets（Lambda関数コード、Dockerイメージ等）の保存
- **ECRリポジトリ**: Dockerイメージの保存
- **IAMロール**: デプロイ時に使用する権限

### クロスアカウントデプロイ
```bash
# ターゲットアカウントでブートストラップ（信頼関係を設定）
cdk bootstrap aws://TARGET_ACCOUNT/REGION \
  --trust PIPELINE_ACCOUNT \
  --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess
```

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS CDK 開発者ガイド](https://docs.aws.amazon.com/cdk/latest/guide/)
- [CDK API リファレンス](https://docs.aws.amazon.com/cdk/api/v2/)
- [CDK Patterns](https://cdkpatterns.com/)
- [CDK Workshop](https://cdkworkshop.com/)
