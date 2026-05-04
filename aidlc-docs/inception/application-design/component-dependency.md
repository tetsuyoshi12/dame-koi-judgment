# コンポーネント依存関係 - ダメ恋プロフィール裁判所

## 依存関係マトリクス

### フロントエンド依存関係

| コンポーネント | 依存先 | 依存理由 |
|--------------|--------|---------|
| App | Phase1Page, Phase2Page, Phase3Page, Phase4Page | Phase表示の制御 |
| App | ApiService | バックエンドAPI呼び出し |
| App | ValidationService | 入力検証 |
| App | LoadingSpinner, ErrorMessage | UI状態表示 |
| Phase1Page | InputField, Button | 入力フォームの構築 |
| Phase2Page | InputField, Slider, Button | 入力フォームの構築 |
| Phase3Page | DarkenTransition, Button | 演出とナビゲーション |
| Phase4Page | StrikethroughAnimation, CopyButton, Button | 演出とコピー機能 |
| CopyButton | ClipboardService | クリップボード操作 |

### バックエンド依存関係

| コンポーネント | 依存先 | 依存理由 |
|--------------|--------|---------|
| AnalyzeHandler | BedrockService | AI分析実行 |
| AnalyzeHandler | PromptService | プロンプト取得 |
| AnalyzeHandler | BackendValidationService | 入力検証 |
| AnalyzeHandler | ErrorService | エラーハンドリング |
| BedrockService | AWS SDK (Bedrock Runtime) | Bedrock API呼び出し |
| PromptService | AWS SDK (SSM) | Parameter Store アクセス |

### インフラ依存関係

| コンポーネント | 依存先 | 依存理由 |
|--------------|--------|---------|
| DameKoiStack | ApiGateway | REST API作成 |
| DameKoiStack | LambdaFunction | Lambda関数作成 |
| DameKoiStack | BedrockPermissions | Bedrock権限設定 |
| DameKoiStack | ParameterStoreConfig | プロンプト保存 |
| ApiGateway | LambdaFunction | Lambda統合 |
| LambdaFunction | BedrockPermissions | Bedrock呼び出し権限 |
| LambdaFunction | ParameterStoreConfig | プロンプト読み取り権限 |

---

## データフロー図

```
[ユーザー]
    |
    | 入力
    v
[Phase1Page] --保存--> [App State]
    |
    | 次へ
    v
[Phase2Page] --保存--> [App State]
    |
    | 送信
    v
[ApiService]
    |
    | POST /api/analyze
    v
[API Gateway]
    |
    | トリガー
    v
[Lambda: AnalyzeHandler]
    |
    +--検証--> [BackendValidationService]
    |
    +--取得--> [PromptService] --> [Parameter Store]
    |
    +--実行--> [BedrockService] --> [Amazon Bedrock]
    |
    | レスポンス
    v
[ApiService]
    |
    | 結果保存
    v
[App State]
    |
    +--表示--> [Phase3Page] --演出--> [DarkenTransition]
    |
    +--表示--> [Phase4Page] --演出--> [StrikethroughAnimation]
                              --コピー--> [CopyButton] --> [ClipboardService]
```

---

## コンポーネント間通信パターン

### 1. フロントエンド内部通信

**パターン**: Props Drilling（シンプルな状態管理）

```typescript
// 親から子へのデータ伝播
App (state)
  └─> Phase1Page (props)
        └─> InputField (props)
              └─> onChange callback
                    └─> App (state update)
```

**通信フロー**:
1. Appコンポーネントが状態を保持
2. 子コンポーネントにpropsとしてデータとコールバックを渡す
3. 子コンポーネントがイベント発生時にコールバックを呼び出し
4. Appコンポーネントが状態を更新

### 2. フロントエンド ↔ バックエンド通信

**パターン**: REST API（JSON over HTTP）

```typescript
// リクエスト
POST /api/analyze
Content-Type: application/json

{
  "phase1": {
    "selfIntro": "...",
    "hobbies": "...",
    "idealPartner": "...",
    "weekendActivities": "..."
  },
  "phase2": {
    "lazyLevel": 4,
    "actualWeekend": "...",
    "lazyHabits": "...",
    "troublesome": "..."
  }
}

// レスポンス
200 OK
Content-Type: application/json

{
  "phase3": {
    "contradictions": ["..."],
    "verdict": "..."
  },
  "phase4": {
    "flawToCharmTranslation": {...},
    "newTarget": "...",
    "newProfile": "...",
    "expectedMatchRate": "..."
  }
}
```

**通信フロー**:
1. ApiServiceがfetch APIでHTTPリクエスト送信
2. API Gatewayがリクエストを受信
3. Lambda関数がトリガーされる
4. Lambda関数が処理を実行してレスポンスを返す
5. ApiServiceがレスポンスをパースして返却

### 3. Lambda ↔ AWS サービス通信

**パターン**: AWS SDK（サービス間通信）

```typescript
// Parameter Store からプロンプト取得
PromptService
  └─> AWS SDK (SSM Client)
        └─> GetParameterCommand
              └─> Parameter Store

// Bedrock API 呼び出し
BedrockService
  └─> AWS SDK (Bedrock Runtime Client)
        └─> InvokeModelCommand
              └─> Amazon Bedrock
```

**通信フロー**:
1. Lambda関数がAWS SDKを使用してサービスを呼び出し
2. IAMロールによる認証・認可
3. サービスが処理を実行してレスポンスを返す
4. Lambda関数がレスポンスを処理

---

## エラー伝播パターン

### フロントエンド エラー伝播

```
[InputField] --validation error--> [Phase1Page] --error state--> [App]
                                                                    |
                                                                    v
                                                              [ErrorMessage]
```

### バックエンド エラー伝播

```
[Bedrock API] --error--> [BedrockService] --throw--> [AnalyzeHandler]
                                                            |
                                                            v
                                                      [ErrorService]
                                                            |
                                                            v
                                                    [Error Response]
                                                            |
                                                            v
                                                      [ApiService]
                                                            |
                                                            v
                                                      [App State]
                                                            |
                                                            v
                                                      [ErrorMessage]
```

---

## 依存関係の方向性

### レイヤー構造

```
+----------------------------------+
|        Presentation Layer        |
|  (Phase Pages, UI Components)    |
+----------------------------------+
              |
              v
+----------------------------------+
|       Application Layer          |
|    (App, Services, State)        |
+----------------------------------+
              |
              v
+----------------------------------+
|      Infrastructure Layer        |
|  (API Gateway, Lambda, Bedrock)  |
+----------------------------------+
```

**依存の原則**:
- 上位レイヤーは下位レイヤーに依存できる
- 下位レイヤーは上位レイヤーに依存しない
- 同一レイヤー内では相互依存を避ける

---

## 循環依存の回避

### 回避策1: コールバック関数の使用

```typescript
// ❌ 循環依存
// Phase1Page が App を import
// App が Phase1Page を import

// ✅ コールバックで回避
// Phase1Page は App を知らない
// App が Phase1Page にコールバックを渡す

interface Phase1PageProps {
  onNext: (data: Phase1Data) => void; // コールバック
}
```

### 回避策2: サービスの分離

```typescript
// ❌ 循環依存
// BedrockService が PromptService を import
// PromptService が BedrockService を import

// ✅ 依存注入で回避
// AnalyzeHandler が両方を管理
class AnalyzeHandler {
  constructor(
    private bedrockService: BedrockService,
    private promptService: PromptService
  ) {}
}
```

---

## 外部依存関係

### NPM パッケージ（フロントエンド）

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "next": "^14.0.0"
  }
}
```

### NPM パッケージ（バックエンド）

```json
{
  "dependencies": {
    "@aws-sdk/client-bedrock-runtime": "^3.x.x",
    "@aws-sdk/client-ssm": "^3.x.x"
  }
}
```

### AWS サービス依存

- **Amazon Bedrock**: Claude モデル（Opus 4.7 または Sonnet 3.5）
- **AWS Systems Manager Parameter Store**: プロンプトテンプレート保存
- **Amazon API Gateway**: REST API
- **AWS Lambda**: サーバーレス関数実行
- **Amazon CloudWatch Logs**: ログ記録

---

## デプロイ依存関係

### デプロイ順序

1. **Parameter Store**: プロンプトテンプレートを先に保存
2. **Lambda + IAM Role**: Lambda関数とBedrockアクセス権限
3. **API Gateway**: Lambda統合
4. **Frontend**: バックエンドAPIのURLを環境変数に設定してデプロイ

### 環境変数依存

**Lambda環境変数**:
- `AWS_REGION`: AWSリージョン
- `BEDROCK_MODEL_ID`: Bedrockモデル ID
- `PROMPT_PARAMETER_NAME`: Parameter Store のパラメータ名

**Frontend環境変数**:
- `NEXT_PUBLIC_API_URL`: バックエンドAPI URL
