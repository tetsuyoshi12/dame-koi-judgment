# Unit of Work 定義 - ダメ恋プロフィール裁判所

## ユニット構成概要

本プロジェクトは、以下の3つのユニットに分解されます：

1. **Frontend Unit** - ユーザーインターフェース
2. **Backend Unit** - ビジネスロジックとAI統合
3. **Infrastructure Unit** - AWSリソース管理とデプロイ

---

## Unit 1: Frontend

### 基本情報

| 項目 | 内容 |
|------|------|
| **ユニット名** | Frontend |
| **技術スタック** | Next.js 14, React 18, TypeScript, CSS Modules/Tailwind CSS |
| **デプロイ先** | Vercel または S3 + CloudFront |
| **開発優先度** | 高（Backend と並行開発） |

### 責任範囲

#### 主要責任
- 4フェーズのユーザーインターフェース実装
- アプリケーション状態管理（useState + props）
- バックエンドAPIとの通信
- UI演出（暗転、抹消線アニメーション）
- クリップボード操作
- クライアント側バリデーション

#### 含まれるコンポーネント
- **ページコンポーネント**: Phase1Page, Phase2Page, Phase3Page, Phase4Page
- **共通UIコンポーネント**: InputField, Slider, Button, LoadingSpinner, ErrorMessage, CopyButton
- **アニメーションコンポーネント**: DarkenTransition, StrikethroughAnimation
- **ルートコンポーネント**: App（状態管理とオーケストレーション）

#### 含まれるサービス
- **ApiService**: バックエンドAPI通信
- **ValidationService**: クライアント側入力検証
- **ClipboardService**: クリップボード操作

### 入力
- ユーザーからの入力（Phase 1, Phase 2データ）
- バックエンドAPIからのレスポンス（Phase 3, Phase 4結果）

### 出力
- レンダリングされたHTML/CSS/JavaScript
- バックエンドAPIへのHTTPリクエスト

### 依存関係
- **外部依存**: Backend Unit（API経由）
- **技術依存**: React, Next.js, TypeScript

### ディレクトリ構造

```
frontend/
├── src/
│   ├── app/
│   │   ├── page.tsx                 # メインページ
│   │   ├── layout.tsx               # レイアウト
│   │   └── globals.css              # グローバルスタイル
│   ├── components/
│   │   ├── pages/
│   │   │   ├── Phase1Page.tsx
│   │   │   ├── Phase2Page.tsx
│   │   │   ├── Phase3Page.tsx
│   │   │   └── Phase4Page.tsx
│   │   ├── ui/
│   │   │   ├── InputField.tsx
│   │   │   ├── Slider.tsx
│   │   │   ├── Button.tsx
│   │   │   ├── LoadingSpinner.tsx
│   │   │   ├── ErrorMessage.tsx
│   │   │   └── CopyButton.tsx
│   │   └── animations/
│   │       ├── DarkenTransition.tsx
│   │       └── StrikethroughAnimation.tsx
│   ├── services/
│   │   ├── ApiService.ts
│   │   ├── ValidationService.ts
│   │   └── ClipboardService.ts
│   └── types/
│       └── index.ts                 # 型定義（shared/から参照）
├── public/
│   └── assets/
├── tests/
│   └── unit/
│       ├── ValidationService.test.ts
│       ├── ClipboardService.test.ts
│       └── ApiService.test.ts
├── package.json
├── tsconfig.json
├── next.config.js
└── .env.local                       # 環境変数（API URL）
```

### 環境変数
- `NEXT_PUBLIC_API_URL`: バックエンドAPI URL

### 開発タスク
1. プロジェクトセットアップ（Next.js初期化）
2. 型定義の作成（shared/から参照）
3. 共通UIコンポーネントの実装
4. Phase 1, 2の入力ページ実装
5. ApiServiceの実装
6. Phase 3, 4の結果表示ページ実装
7. アニメーションコンポーネントの実装
8. 単体テストの作成
9. 統合テスト（手動）

---

## Unit 2: Backend

### 基本情報

| 項目 | 内容 |
|------|------|
| **ユニット名** | Backend |
| **技術スタック** | Node.js 18.x, TypeScript, AWS SDK (Bedrock, SSM) |
| **デプロイ先** | AWS Lambda |
| **開発優先度** | 高（Frontend と並行開発） |

### 責任範囲

#### 主要責任
- API Gatewayからのリクエスト処理
- 入力データのバリデーション
- Amazon Bedrockとの統合
- プロンプトテンプレートの管理
- AI分析結果の生成
- エラーハンドリングとログ記録

#### 含まれるコンポーネント
- **Lambda Handler**: AnalyzeHandler（メインエントリーポイント）
- **サービス**: BedrockService, PromptService, BackendValidationService, ErrorService

### 入力
- API Gatewayからのリクエスト（Phase 1, Phase 2データ）

### 出力
- API Gatewayへのレスポンス（Phase 3, Phase 4結果）
- CloudWatch Logsへのログ

### 依存関係
- **外部依存**: Amazon Bedrock, Parameter Store, CloudWatch Logs
- **技術依存**: AWS SDK, Node.js

### ディレクトリ構造

```
backend/
├── src/
│   ├── handlers/
│   │   └── analyzeHandler.ts       # Lambda ハンドラー
│   ├── services/
│   │   ├── BedrockService.ts
│   │   ├── PromptService.ts
│   │   ├── ValidationService.ts
│   │   └── ErrorService.ts
│   ├── types/
│   │   └── index.ts                 # 型定義（shared/から参照）
│   └── utils/
│       └── logger.ts
├── tests/
│   └── unit/
│       ├── BedrockService.test.ts
│       ├── PromptService.test.ts
│       ├── ValidationService.test.ts
│       └── ErrorService.test.ts
├── package.json
├── tsconfig.json
└── .env                             # ローカル開発用環境変数
```

### 環境変数（Lambda）
- `AWS_REGION`: AWSリージョン（例: ap-northeast-1）
- `BEDROCK_MODEL_ID`: Bedrockモデル ID（例: anthropic.claude-3-5-sonnet-20241022-v2:0）
- `PROMPT_PARAMETER_NAME`: Parameter Store パラメータ名（例: /damekoi/prompt/template）

### 開発タスク
1. プロジェクトセットアップ（TypeScript + AWS SDK）
2. 型定義の作成（shared/から参照）
3. ValidationServiceの実装
4. PromptServiceの実装（Parameter Store連携）
5. BedrockServiceの実装（Bedrock API連携）
6. ErrorServiceの実装
7. AnalyzeHandlerの実装（オーケストレーション）
8. 単体テストの作成
9. ローカルテスト（SAM Local または手動テスト）

---

## Unit 3: Infrastructure

### 基本情報

| 項目 | 内容 |
|------|------|
| **ユニット名** | Infrastructure |
| **技術スタック** | AWS CDK (TypeScript) |
| **デプロイ先** | AWS（API Gateway, Lambda, Parameter Store等） |
| **開発優先度** | 中（Backend/Frontend 完成後） |

### 責任範囲

#### 主要責任
- AWSリソースの定義（IaC）
- API Gatewayの設定
- Lambda関数の設定
- IAMロール・ポリシーの設定
- Parameter Storeの設定
- CloudWatch Logsの設定
- デプロイスクリプトの管理
- 環境変数の管理

#### 含まれるコンポーネント
- **CDKスタック**: DameKoiStack
- **CDKコンストラクト**: ApiGateway, LambdaFunction, BedrockPermissions, ParameterStoreConfig

### 入力
- Backend Unitのビルド成果物（Lambda関数コード）
- プロンプトテンプレート（テキストファイル）

### 出力
- デプロイされたAWSリソース
- API Gateway URL
- CloudFormationスタック

### 依存関係
- **外部依存**: Backend Unit（Lambda関数コード）
- **技術依存**: AWS CDK, AWS CloudFormation

### ディレクトリ構造

```
infrastructure/
├── bin/
│   └── app.ts                       # CDK アプリエントリーポイント
├── lib/
│   ├── stacks/
│   │   └── DameKoiStack.ts          # メインスタック
│   └── constructs/
│       ├── ApiGatewayConstruct.ts
│       ├── LambdaConstruct.ts
│       ├── BedrockPermissionsConstruct.ts
│       └── ParameterStoreConstruct.ts
├── config/
│   ├── prompt-template.txt          # プロンプトテンプレート
│   └── env.ts                       # 環境設定
├── scripts/
│   ├── deploy.sh                    # デプロイスクリプト
│   └── destroy.sh                   # リソース削除スクリプト
├── cdk.json
├── package.json
└── tsconfig.json
```

### 開発タスク
1. CDKプロジェクトセットアップ
2. DameKoiStackの実装
3. API Gateway設定
4. Lambda関数設定（Backend Unitのコードを参照）
5. IAMロール・ポリシー設定
6. Parameter Store設定（プロンプトテンプレート保存）
7. CloudWatch Logs設定
8. デプロイスクリプトの作成
9. デプロイテスト

---

## 共有リソース

### shared/ ディレクトリ

複数ユニットで共有する型定義やユーティリティを管理します。

```
shared/
├── types/
│   ├── phase1.ts                    # Phase 1データ型
│   ├── phase2.ts                    # Phase 2データ型
│   ├── phase3.ts                    # Phase 3結果型
│   ├── phase4.ts                    # Phase 4結果型
│   ├── api.ts                       # API リクエスト/レスポンス型
│   └── index.ts                     # エクスポート
└── openapi/
    └── api-spec.yaml                # OpenAPI 仕様書
```

### OpenAPI 仕様書

Frontend と Backend の並行開発を可能にするため、API仕様を事前に定義します。

**ファイル**: `shared/openapi/api-spec.yaml`

---

## ユニット間の開発順序

### 開発フェーズ

**Phase 1: 準備（Day 1）**
- shared/ の型定義とOpenAPI仕様書を作成
- 各ユニットのプロジェクトセットアップ

**Phase 2: 並行開発（Day 2-11）**
- **Frontend Unit**: UI実装（OpenAPI仕様に基づく）
- **Backend Unit**: ビジネスロジック実装（OpenAPI仕様に基づく）

**Phase 3: 統合（Day 12-13）**
- **Infrastructure Unit**: CDKによるデプロイ
- Frontend と Backend の統合テスト

**Phase 4: 最終調整（Day 14）**
- デモ準備
- バグ修正
- パフォーマンスチューニング

---

## コード組織戦略（Greenfield）

### プロジェクトルート構造

```
dame-koi-judgment/                   # プロジェクトルート
├── frontend/                        # Frontend Unit
├── backend/                         # Backend Unit
├── infrastructure/                  # Infrastructure Unit
├── shared/                          # 共有リソース
├── aidlc-docs/                      # AI-DLC ドキュメント
├── .gitignore
├── README.md
└── package.json                     # ルートpackage.json（monorepo管理用、オプション）
```

### デプロイモデル

- **Frontend**: 独立デプロイ（Vercel または S3+CloudFront）
- **Backend**: Lambda関数として Infrastructure Unit 経由でデプロイ
- **Infrastructure**: CDKによる一括デプロイ

---

## ユニット完成の定義

### Frontend Unit 完成条件
- [ ] 4フェーズのUIがすべて実装されている
- [ ] API通信が正常に動作する
- [ ] アニメーション演出が実装されている
- [ ] クライアント側バリデーションが動作する
- [ ] 主要機能の単体テストがパスする

### Backend Unit 完成条件
- [ ] Lambda関数が正常に動作する
- [ ] Bedrock APIとの統合が完了している
- [ ] プロンプトテンプレートが Parameter Store から取得できる
- [ ] エラーハンドリングが適切に実装されている
- [ ] 主要機能の単体テストがパスする

### Infrastructure Unit 完成条件
- [ ] CDKスタックがデプロイできる
- [ ] API Gateway が正常に動作する
- [ ] Lambda関数が正常にトリガーされる
- [ ] IAM権限が適切に設定されている
- [ ] Parameter Store にプロンプトが保存されている
- [ ] CloudWatch Logs にログが記録される

---

## リスクと対策

### リスク1: Frontend と Backend の統合時の不整合
- **対策**: OpenAPI仕様書を事前に作成し、両ユニットが同じ仕様に従う
- **検証**: 統合テスト時に仕様との整合性を確認

### リスク2: Infrastructure デプロイの遅延
- **対策**: Backend Unit の開発中に Infrastructure Unit の設計を開始
- **検証**: Backend Unit 完成後、速やかにデプロイテスト

### リスク3: 共有型定義の変更による影響
- **対策**: shared/ の型定義変更時は、影響を受けるユニットに通知
- **検証**: TypeScriptのコンパイルエラーで検出
