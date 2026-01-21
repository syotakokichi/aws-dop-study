# AWS Incident Manager

## 概要

AWS Incident Managerは、AWSリソースに影響を与えるインシデントに対して準備し、対応するためのサービス。CloudWatch AlarmsやEventBridgeと統合し、インシデントの検出から解決、事後分析までのライフサイクル全体を管理する。

## キーコンセプト

- **対応計画（Response Plan）**: インシデント発生時のアクションを定義
- **インシデント**: システムに影響を与えるイベント
- **タイムライン**: インシデントの経過を記録
- **エンゲージメント**: オンコール担当者への通知
- **エスカレーション計画**: 段階的な担当者への通知
- **オンコールスケジュール**: 担当者のローテーション
- **ランブック**: インシデント対応の自動化手順（SSM Automation）
- **事後分析（Post-incident Analysis）**: インシデント後の振り返り

## 試験頻出ポイント

- [ ] 対応計画の構成要素（エンゲージメント、ランブック、チャットチャネル）
- [ ] CloudWatch Alarms/EventBridgeからの自動インシデント作成
- [ ] エスカレーション計画の設定
- [ ] SSM Automationランブックとの連携
- [ ] 事後分析レポートの作成
- [ ] オンコールスケジュールの設定
- [ ] PagerDuty/Slack等との連携
- [ ] クロスアカウントインシデント管理

## DOP-C02での出題パターン

### パターン1: アラームからの自動インシデント作成
- **シナリオ**: CloudWatch Alarmがトリガーされたら自動でインシデントを作成
- **ポイント**: 対応計画 + CloudWatch Alarm連携

### パターン2: 段階的エスカレーション
- **シナリオ**: 15分間応答がなければ上位担当者に通知
- **ポイント**: エスカレーション計画のステージ設定

### パターン3: 自動修復
- **シナリオ**: インシデント作成と同時に自動修復ランブックを実行
- **ポイント**: 対応計画にSSM Automationランブックを関連付け

### パターン4: 事後分析
- **シナリオ**: インシデント対応後に改善アクションを特定
- **ポイント**: 事後分析レポートの作成とフォローアップ

## ベストプラクティス

1. **対応計画の事前定義**: 重要なシステムごとに対応計画を作成
2. **ランブックの活用**: 自動修復で対応時間を短縮
3. **エスカレーションの設定**: 応答がない場合の自動エスカレーション
4. **チャットチャネルの統合**: リアルタイムコラボレーション
5. **事後分析の実施**: すべてのインシデントで振り返りを実施
6. **定期的な訓練**: 対応計画のテストと更新

## 対応計画（Response Plan）

### 構成要素

| 要素 | 説明 |
|------|------|
| 表示名 | インシデントのタイトル |
| 影響 | 重大度（1-5） |
| エンゲージメント | 通知する連絡先/エスカレーション計画 |
| チャットチャネル | Slack/Teams等のチャンネル |
| ランブック | 自動実行するSSM Automation |

### 作成例

```yaml
# CloudFormation
ResponsePlan:
  Type: AWS::SSMIncidents::ResponsePlan
  Properties:
    Name: production-incident-plan
    DisplayName: "Production Incident"
    IncidentTemplate:
      Title: "Production Service Disruption"
      Impact: 1  # Critical
      Summary: "Automated incident from CloudWatch Alarm"
    Engagements:
      - !Ref EscalationPlan
    ChatChannel:
      ChatbotSns:
        - !Ref IncidentChatTopic
    Actions:
      - SsmAutomation:
          DocumentName: AWS-RestartEC2Instance
          RoleArn: !GetAtt AutomationRole.Arn
          TargetAccount: RESPONSE_PLAN_OWNER_ACCOUNT
```

## エスカレーション計画

### 構成

```yaml
EscalationPlan:
  Type: AWS::SSMIncidents::EscalationPlan
  Properties:
    Name: on-call-escalation
    Stages:
      - DurationInMinutes: 0
        Targets:
          - ContactTargetInfo:
              ContactId: !Ref OnCallEngineer
              IsEssential: true
      - DurationInMinutes: 15
        Targets:
          - ContactTargetInfo:
              ContactId: !Ref SeniorEngineer
              IsEssential: true
      - DurationInMinutes: 30
        Targets:
          - ContactTargetInfo:
              ContactId: !Ref EngineeringManager
              IsEssential: true
```

### 動作フロー

```
1. インシデント発生（Stage 1）
   → OnCallEngineer に通知
2. 15分間応答なし（Stage 2）
   → SeniorEngineer にエスカレート
3. さらに15分間応答なし（Stage 3）
   → EngineeringManager にエスカレート
```

## 連絡先（Contact）

### 作成

```yaml
OnCallEngineer:
  Type: AWS::SSMContacts::Contact
  Properties:
    Alias: on-call-engineer
    DisplayName: "On-Call Engineer"
    Type: PERSONAL
    Plan:
      - DurationInMinutes: 0
        Targets:
          - ChannelTargetInfo:
              ContactChannelId: !Ref EmailChannel
              RetryIntervalInMinutes: 5
          - ChannelTargetInfo:
              ContactChannelId: !Ref SMSChannel
              RetryIntervalInMinutes: 5

EmailChannel:
  Type: AWS::SSMContacts::ContactChannel
  Properties:
    ContactId: !Ref OnCallEngineer
    ChannelName: Work Email
    ChannelType: EMAIL
    ChannelAddress: oncall@example.com
```

## ランブックとの連携

### 自動修復の例

```yaml
# 対応計画でランブックを指定
Actions:
  - SsmAutomation:
      DocumentName: Custom-RestartService
      DocumentVersion: "1"
      RoleArn: !GetAtt AutomationRole.Arn
      Parameters:
        ServiceName:
          - "{{ INCIDENT.TITLE }}"
```

### SSM Automationドキュメント例

```yaml
# 自動修復ランブック
schemaVersion: '0.3'
description: 'Restart unhealthy service'
parameters:
  ServiceName:
    type: String
mainSteps:
  - name: GetUnhealthyInstances
    action: aws:executeScript
    outputs:
      - Name: InstanceIds
        Selector: $.Payload.instanceIds
        Type: StringList
    inputs:
      Runtime: python3.8
      Handler: script_handler
      Script: |
        # 異常インスタンスを特定するロジック

  - name: RestartInstances
    action: aws:changeInstanceState
    inputs:
      InstanceIds: '{{ GetUnhealthyInstances.InstanceIds }}'
      DesiredState: running
```

## CloudWatch Alarm連携

### 設定

```yaml
HighCPUAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: HighCPU
    MetricName: CPUUtilization
    Namespace: AWS/EC2
    Statistic: Average
    Period: 300
    EvaluationPeriods: 2
    Threshold: 90
    ComparisonOperator: GreaterThanThreshold
    AlarmActions:
      - !GetAtt ResponsePlan.Arn  # Incident Manager対応計画
```

## インシデントライフサイクル

```
1. 検出（Detection）
   - CloudWatch Alarm / EventBridge / 手動作成

2. 対応開始（Engagement）
   - 対応計画に基づいて担当者に通知
   - ランブック自動実行
   - チャットチャネルでコラボレーション

3. 調査・対応（Investigation）
   - タイムラインに経過を記録
   - 関連メトリクス/ログの確認
   - 追加アクションの実行

4. 解決（Resolution）
   - インシデントをクローズ
   - 解決サマリーを記録

5. 事後分析（Post-incident Analysis）
   - タイムラインのレビュー
   - 改善アクションの特定
   - フォローアップアイテムの作成
```

## 事後分析

### 含める内容

- インシデントの概要とタイムライン
- 影響範囲（影響を受けた顧客数、期間等）
- 根本原因分析（RCA）
- 対応で良かった点
- 改善が必要な点
- フォローアップアクションアイテム

### ベストプラクティス

1. **blame-freeな文化**: 個人を責めず、システム改善に焦点
2. **5つのなぜ**: 根本原因を深掘り
3. **アクションアイテムの追跡**: 改善が実施されるまでフォロー

## 他サービスとの連携

| サービス | 連携内容 |
|----------|----------|
| CloudWatch Alarms | インシデント自動作成のトリガー |
| EventBridge | イベント駆動でインシデント作成 |
| Systems Manager | ランブック自動実行 |
| SNS | エンゲージメント通知 |
| AWS Chatbot | Slack/Teams連携 |
| PagerDuty | 外部オンコール管理 |
| Organizations | クロスアカウント管理 |

## ハンズオン記録

<!-- 実施後に記入 -->

## 参考リンク

- [AWS Incident Manager ユーザーガイド](https://docs.aws.amazon.com/incident-manager/latest/userguide/)
- [対応計画の作成](https://docs.aws.amazon.com/incident-manager/latest/userguide/response-plans.html)
- [エスカレーション計画](https://docs.aws.amazon.com/incident-manager/latest/userguide/escalation.html)
