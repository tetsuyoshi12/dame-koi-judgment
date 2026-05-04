# アプリケーション設計 統合ドキュメント

## 設計概要

「ダメ恋プロフィール裁判所」は、React SPA フロントエンドと Python Lambda バックエンドで構成される単一ユニットの Web アプリケーション。Amazon Bedrock（Claude Opus 4.7）を AI エンジンとして使用し、4フェーズのユーザーフローを通じて「立派な人を演じる努力を奪い、ダメなまま勝つ戦略を発行する」体験を提供する。

---

## アーキテクチャ全体像

```
+------------------------------------------------------------------+
|                    ユーザーブラウザ                               |
|  +------------------------------------------------------------+  |
|  |              React SPA (S3 + CloudFront)                   |  |
|  |                                                            |  |
|  |  [App]                                                     |  |
|  |    |-- [Phase1Page]  虚飾入力フォーム                       |  |
|  |    |-- [Phase2Page]  本音質問 (動的生成)                    |  |
|  |    |-- [Phase3Page]  暗転 + 否定文演出                      |  |
|  |    |-- [Phase4Page]  抹消線 + 改善案発行                    |  |
|  |    |-- [MonolithCharacter]  AIキャラクター                  |  |
|  |                                                            |  |
|  |  Services: SessionOrchestration / ApiClient / Animation    |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
                              |
                              | HTTPS (REST)
                              v
+------------------------------------------------------------------+
|                    API Gateway (ap-northeast-1)                  |
|  POST /generate-questions                                        |
|  POST /analyze                                                   |
+------------------------------------------------------------------+
          |                              |
          v                              v
+------------------+          +------------------+
| GenerateQuestions|          |  AnalyzeLambda   |
|     Lambda       |          |  (Python)        |
|  (Python)        |          |                  |
|                  |          |  BedrockPrompt   |
|  BedrockPrompt   |          |  Service         |
|  Service         |          |  SessionStorage  |
+------------------+          |  Service         |
          |                   +------------------+
          |                              |
          v                              v
+------------------------------------------------------------------+
|                    AWS サービス                                   |
|                                                                  |
|  +------------------+    +----------------------------------+    |
|  | Amazon Bedrock   |    | S3 (データバケット)               |    |
|  | Claude Opus 4.7  |    |  prompts/ (テンプレート)          |    |
|  | ap-northeast-1   |    |  sessions/ (匿名データ)           |    |
|  +------------------+    +----------------------------------+    |
+------------------------------------------------------------------+
```

---

## コンポーネント一覧

### フロントエンド（React SPA）
| ID | コンポーネント | 種別 | 詳細 |
|----|--------------|------|------|
| FE-01 | App | ページ | ルート・Redux Store プロバイダー |
| FE-02 | Phase1Page | ページ | 虚飾プロフィール入力フォーム |
| FE-03 | Phase2Page | ページ | 動的本音質問表示・回答入力 |
| FE-04 | Phase3Page | ページ | 暗転演出・否定文表示 |
| FE-05 | Phase4Page | ページ | 抹消線演出・改善案発行 |
| FE-06 | MonolithCharacter | UI | AIキャラクタービジュアル |

### フロントエンドサービス
| ID | サービス | 役割 |
|----|---------|------|
| SVC-01 | SessionOrchestrationService | Redux Store による全フェーズデータ管理 |
| SVC-02 | ApiClientService | バックエンド API 通信の抽象化 |
| SVC-03 | AnimationOrchestrationService | 演出アニメーションシーケンス制御 |

### バックエンド（Python Lambda）
| ID | コンポーネント | 種別 | 詳細 |
|----|--------------|------|------|
| BE-01 | GenerateQuestionsLambda | Lambda | Phase 1 → Phase 2 質問生成 |
| BE-02 | AnalyzeLambda | Lambda | Phase 1/2 → Phase 3/4 分析・改善案生成 |

### バックエンドサービス
| ID | サービス | 役割 |
|----|---------|------|
| SVC-04 | BedrockPromptService | プロンプト構築・Bedrock 呼び出し |
| SVC-05 | SessionStorageService | 匿名セッションデータの S3 保存 |

### AWSインフラ
| ID | リソース | 用途 |
|----|---------|------|
| INFRA-01 | API Gateway | HTTP ルーティング・CORS |
| INFRA-02 | S3（データバケット） | プロンプトテンプレート + セッションデータ |
| INFRA-03 | S3 + CloudFront | React SPA ホスティング |
| INFRA-04 | Amazon Bedrock | Claude Opus 4.7 AI エンジン |

---

## 主要設計決定

| 決定事項 | 選択 | 理由 |
|---------|------|------|
| フロントエンド FW | React SPA | シンプルな構成、ハッカソン向け |
| 状態管理 | Redux Toolkit | 4フェーズをまたぐ複雑な状態管理に適切 |
| Lambda 構成 | 機能別 2 Lambda | 責務分離・タイムアウト設定の独立管理 |
| Bedrock 呼び出し | 同期 | シンプルな実装、15秒以内で収まる見込み |
| データ保存 | S3（JSON） | シンプル・低コスト、ハッカソン用途に最適 |
| プロンプト管理 | S3 外部化 | 再デプロイなしでプロンプトチューニング可能 |
| アニメーション | Pure CSS | 外部ライブラリ依存なし、軽量 |
| ホスティング | S3 + CloudFront | コスト最小、CDN による高速配信 |

---

## ユーザーストーリー → コンポーネント対応

| ストーリー | 主要コンポーネント |
|-----------|----------------|
| US-01 虚飾入力 | Phase1Page |
| US-02 フォームUX | Phase1Page |
| US-03 本音質問表示 | Phase2Page + GenerateQuestionsLambda + BedrockPromptService |
| US-04 本音回答入力 | Phase2Page + SessionOrchestrationService |
| US-05 暗転演出 | Phase3Page + AnimationOrchestrationService |
| US-06 学術的否定 | Phase3Page + AnalyzeLambda + BedrockPromptService |
| US-07 ダメさ翻訳・新プロフィール | Phase4Page + AnalyzeLambda |
| US-08 抹消線演出 | Phase4Page + AnimationOrchestrationService |
| US-09 マッチング率表示 | Phase4Page |
