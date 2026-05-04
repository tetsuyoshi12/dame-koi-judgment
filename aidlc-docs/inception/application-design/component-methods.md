# コンポーネントメソッド定義 - ダメ恋プロフィール裁判所

## フロントエンドコンポーネントメソッド

### App（ルートコンポーネント）

```typescript
interface AppState {
  currentPhase: 1 | 2 | 3 | 4;
  phase1Data: Phase1Data;
  phase2Data: Phase2Data;
  analysisResult: AnalysisResult | null;
  isLoading: boolean;
  error: string | null;
}

// Phase遷移
goToPhase(phase: number): void

// Phase 1データの保存
savePhase1Data(data: Phase1Data): void

// Phase 2データの保存と分析実行
savePhase2DataAndAnalyze(data: Phase2Data): Promise<void>

// バックエンドAPI呼び出し
callAnalyzeAPI(phase1: Phase1Data, phase2: Phase2Data): Promise<AnalysisResult>

// エラーハンドリング
handleError(error: Error): void

// エラークリア
clearError(): void
```

### Phase1Page

```typescript
interface Phase1Data {
  selfIntro: string;
  hobbies: string;
  idealPartner: string;
  weekendActivities: string;
}

// 入力値の変更
handleInputChange(field: keyof Phase1Data, value: string): void

// バリデーション
validateInputs(): boolean

// Phase 2への遷移
handleNext(): void
```

### Phase2Page

```typescript
interface Phase2Data {
  lazyLevel: number;
  actualWeekend: string;
  lazyHabits: string;
  troublesome: string;
}

// 入力値の変更
handleInputChange(field: keyof Phase2Data, value: string | number): void

// バリデーション
validateInputs(): boolean

// 分析実行
handleSubmit(): Promise<void>
```

### Phase3Page

```typescript
interface Phase3Result {
  contradictions: string[];
  verdict: string;
}

// Phase 4への遷移
handleNext(): void

// 暗転演出の制御
triggerDarkenAnimation(): void
```

### Phase4Page

```typescript
interface Phase4Result {
  flawToCharmTranslation: Record<string, string>;
  newTarget: string;
  newProfile: string;
  expectedMatchRate: string;
}

// 新プロフィール文のコピー
handleCopyProfile(): void

// コピー成功時の処理
onCopySuccess(): void

// 抹消線アニメーションの制御
triggerStrikethroughAnimation(): void

// 最初からやり直す
handleRestart(): void
```

### InputField

```typescript
interface InputFieldProps {
  label: string;
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  error?: string;
  maxLength?: number;
  rows?: number;
}

// 入力値の変更
handleChange(event: React.ChangeEvent<HTMLTextAreaElement>): void
```

### Slider

```typescript
interface SliderProps {
  label: string;
  value: number;
  onChange: (value: number) => void;
  min: number;
  max: number;
  step?: number;
}

// スライダー値の変更
handleChange(event: React.ChangeEvent<HTMLInputElement>): void

// 値のラベル表示
getValueLabel(value: number): string
```

### Button

```typescript
interface ButtonProps {
  text: string;
  onClick: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary';
  isLoading?: boolean;
}

// クリックイベント
handleClick(): void
```

### CopyButton

```typescript
interface CopyButtonProps {
  text: string;
  onCopySuccess?: () => void;
}

// クリップボードへのコピー
copyToClipboard(): Promise<void>

// コピー成功時の処理
handleCopySuccess(): void

// コピー失敗時の処理
handleCopyError(error: Error): void
```

### DarkenTransition

```typescript
interface DarkenTransitionProps {
  isActive: boolean;
  children: React.ReactNode;
  duration?: number;
}

// アニメーション開始
startAnimation(): void

// アニメーション完了
onAnimationComplete(): void
```

### StrikethroughAnimation

```typescript
interface StrikethroughAnimationProps {
  oldText: string;
  newText: string;
  isActive: boolean;
}

// 抹消線アニメーション開始
startStrikethrough(): void

// 新テキスト表示
showNewText(): void

// アニメーション完了
onAnimationComplete(): void
```

---

## バックエンドコンポーネントメソッド

### AnalyzeHandler（Lambda関数）

```typescript
interface AnalyzeRequest {
  phase1: Phase1Data;
  phase2: Phase2Data;
}

interface AnalyzeResponse {
  phase3: Phase3Result;
  phase4: Phase4Result;
}

// Lambda ハンドラー
handler(event: APIGatewayProxyEvent, context: Context): Promise<APIGatewayProxyResult>

// リクエストボディのパース
parseRequestBody(body: string): AnalyzeRequest

// リクエストの検証
validateRequest(request: AnalyzeRequest): void

// 分析処理の実行
executeAnalysis(request: AnalyzeRequest): Promise<AnalyzeResponse>

// レスポンスの生成
createSuccessResponse(data: AnalyzeResponse): APIGatewayProxyResult

// エラーレスポンスの生成
createErrorResponse(error: Error): APIGatewayProxyResult
```

### BedrockService

```typescript
interface BedrockConfig {
  modelId: string;
  region: string;
  maxTokens: number;
  temperature: number;
}

// プロフィール分析の実行
analyzeProfile(phase1: Phase1Data, phase2: Phase2Data): Promise<AnalysisResult>

// プロンプトの構築
buildPrompt(phase1: Phase1Data, phase2: Phase2Data, template: string): string

// Bedrock APIの呼び出し
invokeBedrock(prompt: string): Promise<string>

// レスポンスのパース
parseResponse(response: string): AnalysisResult

// タイムアウト処理
handleTimeout(): void
```

### PromptManager

```typescript
// プロンプトテンプレートの取得
getPromptTemplate(): Promise<string>

// Parameter Storeからの取得
fetchFromParameterStore(parameterName: string): Promise<string>

// プロンプトへのデータ埋め込み
fillPromptData(template: string, phase1: Phase1Data, phase2: Phase2Data): string

// プレースホルダーの置換
replacePlaceholders(template: string, data: Record<string, string>): string
```

### ValidationService

```typescript
// Phase 1データの検証
validatePhase1Data(data: Phase1Data): ValidationResult

// Phase 2データの検証
validatePhase2Data(data: Phase2Data): ValidationResult

// 必須項目チェック
checkRequiredFields(data: object, requiredFields: string[]): boolean

// 文字列長チェック
checkStringLength(value: string, maxLength: number): boolean

// 数値範囲チェック
checkNumberRange(value: number, min: number, max: number): boolean
```

### ErrorHandler

```typescript
// エラーのログ記録
logError(error: Error, context: Record<string, any>): void

// CloudWatch Logsへの記録
writeToCloudWatch(logMessage: string): void

// エラーレスポンスの生成
createErrorResponse(error: Error, statusCode: number): APIGatewayProxyResult

// エラー種別の判定
determineErrorType(error: Error): ErrorType

// ユーザー向けエラーメッセージの生成
generateUserMessage(errorType: ErrorType): string
```

---

## インフラコンポーネントメソッド

### DameKoiStack（CDKスタック）

```typescript
// コンストラクタ
constructor(scope: Construct, id: string, props?: StackProps)

// API Gatewayの作成
createApiGateway(): RestApi

// Lambda関数の作成
createLambdaFunction(): Function

// IAMロールの作成
createLambdaRole(): Role

// Parameter Storeの設定
setupParameterStore(): void

// CloudWatch Logsの設定
setupCloudWatchLogs(): void

// リソース間の接続
connectResources(): void
```

### ApiGateway（CDKコンストラクト）

```typescript
// REST APIの作成
createRestApi(name: string): RestApi

// リソースの追加
addResource(path: string): Resource

// メソッドの追加
addMethod(resource: Resource, method: string, integration: LambdaIntegration): Method

// CORSの設定
configureCors(resource: Resource): void

// レート制限の設定
configureRateLimit(method: Method): void
```

### LambdaFunction（CDKコンストラクト）

```typescript
// Lambda関数の作成
createFunction(props: FunctionProps): Function

// 環境変数の設定
setEnvironmentVariables(func: Function, vars: Record<string, string>): void

// タイムアウトの設定
setTimeout(func: Function, seconds: number): void

// メモリの設定
setMemory(func: Function, megabytes: number): void

// レイヤーの追加
addLayer(func: Function, layer: LayerVersion): void
```

### BedrockPermissions（CDKコンストラクト）

```typescript
// Bedrock権限の付与
grantBedrockAccess(role: Role): void

// IAMポリシーの作成
createBedrockPolicy(): PolicyStatement

// ロールへのポリシー追加
attachPolicyToRole(role: Role, policy: PolicyStatement): void
```

### ParameterStoreConfig（CDKコンストラクト）

```typescript
// パラメータの作成
createParameter(name: string, value: string): StringParameter

// パラメータの取得権限付与
grantReadAccess(role: Role, parameter: StringParameter): void

// プロンプトテンプレートの保存
savePromptTemplate(template: string): void
```

---

## 型定義

```typescript
// Phase 1データ
interface Phase1Data {
  selfIntro: string;
  hobbies: string;
  idealPartner: string;
  weekendActivities: string;
}

// Phase 2データ
interface Phase2Data {
  lazyLevel: number;
  actualWeekend: string;
  lazyHabits: string;
  troublesome: string;
}

// Phase 3結果
interface Phase3Result {
  contradictions: string[];
  verdict: string;
}

// Phase 4結果
interface Phase4Result {
  flawToCharmTranslation: Record<string, string>;
  newTarget: string;
  newProfile: string;
  expectedMatchRate: string;
}

// 分析結果
interface AnalysisResult {
  phase3: Phase3Result;
  phase4: Phase4Result;
}

// バリデーション結果
interface ValidationResult {
  isValid: boolean;
  errors: string[];
}

// エラータイプ
enum ErrorType {
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  BEDROCK_ERROR = 'BEDROCK_ERROR',
  TIMEOUT_ERROR = 'TIMEOUT_ERROR',
  INTERNAL_ERROR = 'INTERNAL_ERROR',
}
```
