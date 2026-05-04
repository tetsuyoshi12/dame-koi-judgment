# Application Design - ダメ恋プロフィール裁判所

## 設計概要

本ドキュメントは、「ダメ恋プロフィール裁判所」のアプリケーション設計を統合的にまとめたものです。

---

## アーキテクチャ概要

### システム構成

```
+------------------+
|   ユーザー        |
+------------------+
         |
         | HTTPS
         v
+------------------+
|  Frontend        |
|  (Next.js)       |
+------------------+
         |
         | REST API
         v
+------------------+
|  API Gateway     |
+------------------+
         |
         | Trigger
         v
+------------------+
|  Lambda          |
|  (Node.js)       |
+------------------+
         |
         +---> Parameter Store (プロンプト)
         |
         +---> Amazon Bedrock (Claude)
         |
         +---> CloudWatch Logs
```

### 技術スタック

**フロントエンド**:
- Next.js 14
- React 18
- TypeScript
- CSS Modules / Tailwind CSS

**バックエンド**:
- AWS Lambda (Node.js 18.x)
- Amazon Bedrock (Claude Opus 4.7 / Sonnet 3.5)
- AWS Systems Manager Parameter Store
- Amazon API Gateway (REST API)

**インフラ**:
- AWS CDK (TypeScript)
- Amazon CloudWatch Logs

---

## コンポーネント設計

### フロントエンドコンポーネント階層

```
App (ルート)
├── Phase1Page (虚飾の入力)
│   ├── InputField (自己紹介文)
│   ├── InputField (趣味・特技)
│   ├── InputField (理想の相手像)
│   ├── InputField (休日の過ごし方)
│   └── Button (次へ)
│
├── Phase2Page (本音の入力)
│   ├── Slider (ズボラ度)
│   ├── InputField (実際の休日)
│   ├── InputField (サボり癖)
│   ├── InputField (面倒なこと)
│   └── Button (分析実行)
│
├── Phase3Page (論理的否定)
│   ├── DarkenTransition (暗転演出)
│   │   └── 否定文表示
│   └── Button (次へ)
│
├── Phase4Page (改善案発行)
│   ├── StrikethroughAnimation (抹消線演出)
│   ├── 改善案表示
│   ├── CopyButton (コピー)
│   └── Button (最初から)
│
├── LoadingSpinner (ローディング)
└── ErrorMessage (エラー表示)
```

### バックエンドコンポーネント構成

```
AnalyzeHandler (Lambda関数)
├── BackendValidationService (入力検証)
├── PromptService (プロンプト管理)
│   └── Parameter Store 連携
├── BedrockService (AI分析)
│   └── Bedrock API 連携
└── ErrorService (エラーハンドリング)
    └── CloudWatch Logs 連携
```

---

## データモデル

### Phase 1 データ（虚飾の入力）

```typescript
interface Phase1Data {
  selfIntro: string;          // 自己紹介文
  hobbies: string;            // 趣味・特技
  idealPartner: string;       // 理想の相手像
  weekendActivities: string;  // 休日の過ごし方
}
```

### Phase 2 データ（本音の入力）

```typescript
interface Phase2Data {
  lazyLevel: number;          // ズボラ度 (1-5)
  actualWeekend: string;      // 実際の休日の過ごし方
  lazyHabits: string;         // サボり癖・怠惰な習慣
  troublesome: string;        // 本当は面倒だと思っていること
}
```

### Phase 3 結果（論理的否定）

```typescript
interface Phase3Result {
  contradictions: string[];   // 矛盾の指摘（配列）
  verdict: string;            // 判決文
}
```

### Phase 4 結果（改善案発行）

```typescript
interface Phase4Result {
  flawToCharmTranslation: Record<string, string>;  // ダメさ→魅力の翻訳
  newTarget: string;                               // 新ターゲット層
  newProfile: string;                              // 新プロフィール文
  expectedMatchRate: string;                       // 期待マッチング率
}
```

### 分析結果（統合）

```typescript
interface AnalysisResult {
  phase3: Phase3Result;
  phase4: Phase4Result;
}
```

---

## API設計

### エンドポイント

**POST /api/analyze**

**リクエスト**:
```json
{
  "phase1": {
    "selfIntro": "週末はカフェ巡りやジムでリフレッシュ...",
    "hobbies": "読書、映画鑑賞、新しいレストランの開拓",
    "idealPartner": "互いに高め合える関係が理想",
    "weekendActivities": "アクティブに過ごしています"
  },
  "phase2": {
    "lazyLevel": 4,
    "actualWeekend": "カップ麺とYouTube",
    "lazyHabits": "ジムは月1回、本当は何もしたくない",
    "troublesome": "外出、人と会うこと、自己アピール"
  }
}
```

**レスポンス（成功）**:
```json
{
  "phase3": {
    "contradictions": [
      "被告は『アクティブな休日』を標榜しながら、自己申告の実態は『カップ麺とYouTube』。行動心理学における自己呈示理論の典型的な失敗例である。",
      "『ジムでリフレッシュ』と記述しながら、実際の利用頻度は月1回。統計的に見て、この頻度では健康効果は期待できず、虚飾と判断される。"
    ],
    "verdict": "以上により、被告に『立派な人を演じる義務』からの完全免除を言い渡す。"
  },
  "phase4": {
    "flawToCharmTranslation": {
      "ズボラ": "一緒にいて楽な人",
      "インドア派": "落ち着いた時間を共有できる",
      "面倒くさがり": "無理しない関係を築ける"
    },
    "newTarget": "等身大の生活を共有したい、無理しない関係を求める層（25-35歳、同じくインドア派）",
    "newProfile": "休日は基本インドア派。カップ麺とYouTubeで満足できる、飾らない性格です。一緒にダラダラできる、気を使わない関係が理想。無理に外出するより、家で映画を見ながらゴロゴロする方が好きです。",
    "expectedMatchRate": "従来プロフィール: 3.2% → 新プロフィール: 18.7%（推定）"
  }
}
```

**レスポンス（エラー）**:
```json
{
  "error": "入力データが不正です"
}
```

**ステータスコード**:
- 200: 成功
- 400: バリデーションエラー
- 500: サーバーエラー
- 502: Bedrockエラー
- 504: タイムアウト

---

## 状態管理設計

### App State

```typescript
interface AppState {
  currentPhase: 1 | 2 | 3 | 4;           // 現在のPhase
  phase1Data: Phase1Data | null;         // Phase 1データ
  phase2Data: Phase2Data | null;         // Phase 2データ
  analysisResult: AnalysisResult | null; // 分析結果
  isLoading: boolean;                    // ローディング状態
  error: string | null;                  // エラーメッセージ
}
```

### 状態遷移

```
[Phase 1] --入力完了--> [Phase 2] --分析実行--> [Loading]
                                                    |
                                                    v
                                              [Phase 3] --次へ--> [Phase 4]
                                                                       |
                                                                       v
                                                                  [最初から] --> [Phase 1]
```

---

## UI/UX設計

### Phase 1: 虚飾の入力

**レイアウト**:
- タイトル: 「あなたの理想のプロフィールを入力してください」
- 4つのテキストエリア（各500文字まで）
- 次へボタン

**バリデーション**:
- 全項目必須
- 文字数制限チェック

### Phase 2: 本音の入力

**レイアウト**:
- タイトル: 「本当のあなたを教えてください」
- ズボラ度スライダー（1-5）
- 3つのテキストエリア
- 分析実行ボタン

**バリデーション**:
- 全項目必須
- スライダー範囲チェック

### Phase 3: 論理的否定

**演出**:
1. 画面暗転（2秒）
2. 無音状態
3. 否定文が浮かび上がる（フェードイン）
4. 判決文表示

**レイアウト**:
- 冷徹な裁判官トーンの文体
- 矛盾の指摘（箇条書き）
- 判決文（太字）
- 次へボタン

### Phase 4: 改善案発行

**演出**:
1. Phase 1の入力文に抹消線（アニメーション）
2. 新プロフィール文が上書き表示

**レイアウト**:
- ダメさ→魅力の翻訳表
- 新ターゲット層の説明
- 新プロフィール文（ハイライト）
- 期待マッチング率（グラフ表示）
- コピーボタン
- 最初からボタン

---

## セキュリティ設計

### フロントエンド

- **入力検証**: クライアント側で基本的なバリデーション
- **XSS対策**: Reactの自動エスケープ機能を利用
- **HTTPS**: 本番環境では必須

### バックエンド

- **入力検証**: サーバー側で厳密なバリデーション
- **IAM権限**: 最小権限の原則（Bedrockへのアクセスのみ）
- **エラーハンドリング**: 内部エラーの詳細を外部に漏らさない
- **ログ記録**: CloudWatch Logsにエラーログを記録

### API

- **CORS**: 許可されたオリジンのみ
- **レート制限**: API Gatewayで設定（オプション）
- **認証**: ハッカソンデモでは不要（将来的にはCognito検討）

---

## パフォーマンス設計

### フロントエンド

- **コード分割**: Next.jsの自動コード分割
- **画像最適化**: Next.js Image コンポーネント
- **キャッシング**: ブラウザキャッシュ活用

### バックエンド

- **Lambda最適化**:
  - メモリ: 512MB
  - タイムアウト: 30秒
  - コールドスタート対策: Provisioned Concurrency（オプション）

- **Bedrock最適化**:
  - プロンプトトークン数の最小化
  - max_tokens: 4000
  - temperature: 0.7

### 目標値

- **フロントエンド初期表示**: 3秒以内
- **Bedrockレスポンス**: 15秒以内
- **合計処理時間**: 20秒以内

---

## エラーハンドリング設計

### エラー種別

| エラータイプ | 原因 | ユーザーメッセージ | ステータスコード |
|------------|------|------------------|----------------|
| VALIDATION_ERROR | 入力データ不正 | 入力データが不正です | 400 |
| TIMEOUT_ERROR | Bedrockタイムアウト | サーバーの応答がタイムアウトしました | 504 |
| BEDROCK_ERROR | Bedrock API エラー | AI分析サービスでエラーが発生しました | 502 |
| INTERNAL_ERROR | その他のエラー | 予期しないエラーが発生しました | 500 |

### エラー処理フロー

```
[エラー発生]
    |
    v
[ErrorService.logError()] --> CloudWatch Logs
    |
    v
[ErrorService.createErrorResponse()]
    |
    v
[ユーザーに汎用メッセージ表示]
```

---

## テスト戦略

### フロントエンド

- **単体テスト**: 主要コンポーネントのロジック
  - ValidationService
  - ClipboardService
  - ApiService

### バックエンド

- **単体テスト**: 主要サービスクラス
  - BedrockService
  - PromptService
  - BackendValidationService
  - ErrorService

### 統合テスト

- **E2Eテスト**: 4フェーズの完全なフロー（手動テスト）

---

## デプロイ設計

### デプロイ環境

- **Frontend**: Vercel または S3 + CloudFront
- **Backend**: AWS Lambda（CDKでデプロイ）

### デプロイ手順

1. **Parameter Store設定**: プロンプトテンプレートを保存
2. **CDKデプロイ**: `cdk deploy` でバックエンドをデプロイ
3. **環境変数設定**: フロントエンドにAPI URLを設定
4. **フロントエンドデプロイ**: Vercelまたはs3にデプロイ
5. **動作確認**: E2Eテスト実行

### 環境変数

**Lambda**:
- `AWS_REGION`: ap-northeast-1
- `BEDROCK_MODEL_ID`: anthropic.claude-opus-4-7 または anthropic.claude-3-5-sonnet-20241022-v2:0
- `PROMPT_PARAMETER_NAME`: /damekoi/prompt/template

**Frontend**:
- `NEXT_PUBLIC_API_URL`: https://xxxxxxxx.execute-api.ap-northeast-1.amazonaws.com/prod

---

## 今後の拡張性

### 短期的な拡張

- **複数トーン対応**: ユーザーがトーンを選択できる
- **結果の保存**: DynamoDBに保存して履歴表示
- **SNSシェア**: 結果をTwitter等でシェア

### 長期的な拡張

- **認証機能**: Amazon Cognito導入
- **マルチテナント**: 複数ユーザーの管理
- **A/Bテスト**: プロンプトの効果測定
- **分析ダッシュボード**: 利用統計の可視化

---

## 参照ドキュメント

- [components.md](./components.md) - コンポーネント定義
- [component-methods.md](./component-methods.md) - メソッドシグネチャ
- [services.md](./services.md) - サービス定義
- [component-dependency.md](./component-dependency.md) - 依存関係
- [requirements.md](../requirements/requirements.md) - 要件定義
