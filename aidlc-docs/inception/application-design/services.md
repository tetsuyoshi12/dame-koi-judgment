# サービス定義とオーケストレーション - ダメ恋プロフィール裁判所

## サービス層の概要

サービス層は、ビジネスロジックの実行とコンポーネント間のオーケストレーションを担当します。

---

## フロントエンドサービス

### 1. ApiService
**責任範囲**: バックエンドAPIとの通信を統括

**メソッド**:

```typescript
class ApiService {
  private baseUrl: string;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  // プロフィール分析API呼び出し
  async analyzeProfile(
    phase1: Phase1Data, 
    phase2: Phase2Data
  ): Promise<AnalysisResult> {
    const response = await fetch(`${this.baseUrl}/api/analyze`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ phase1, phase2 }),
    });
    
    if (!response.ok) {
      throw new Error(`API Error: ${response.status}`);
    }
    
    return await response.json();
  }
  
  // エラーハンドリング
  handleApiError(error: Error): string {
    if (error.message.includes('timeout')) {
      return 'サーバーの応答がタイムアウトしました。もう一度お試しください。';
    }
    return '予期しないエラーが発生しました。しばらくしてからお試しください。';
  }
}
```

### 2. ValidationService
**責任範囲**: フロントエンド側の入力検証

**メソッド**:

```typescript
class ValidationService {
  // Phase 1データの検証
  validatePhase1(data: Phase1Data): ValidationResult {
    const errors: string[] = [];
    
    if (!data.selfIntro || data.selfIntro.trim().length === 0) {
      errors.push('自己紹介文を入力してください');
    }
    if (data.selfIntro && data.selfIntro.length > 500) {
      errors.push('自己紹介文は500文字以内で入力してください');
    }
    
    if (!data.hobbies || data.hobbies.trim().length === 0) {
      errors.push('趣味・特技を入力してください');
    }
    
    if (!data.idealPartner || data.idealPartner.trim().length === 0) {
      errors.push('理想の相手像を入力してください');
    }
    
    if (!data.weekendActivities || data.weekendActivities.trim().length === 0) {
      errors.push('休日の過ごし方を入力してください');
    }
    
    return {
      isValid: errors.length === 0,
      errors,
    };
  }
  
  // Phase 2データの検証
  validatePhase2(data: Phase2Data): ValidationResult {
    const errors: string[] = [];
    
    if (data.lazyLevel < 1 || data.lazyLevel > 5) {
      errors.push('ズボラ度は1〜5の範囲で選択してください');
    }
    
    if (!data.actualWeekend || data.actualWeekend.trim().length === 0) {
      errors.push('実際の休日の過ごし方を入力してください');
    }
    
    if (!data.lazyHabits || data.lazyHabits.trim().length === 0) {
      errors.push('サボり癖・怠惰な習慣を入力してください');
    }
    
    if (!data.troublesome || data.troublesome.trim().length === 0) {
      errors.push('本当は面倒だと思っていることを入力してください');
    }
    
    return {
      isValid: errors.length === 0,
      errors,
    };
  }
}
```

### 3. ClipboardService
**責任範囲**: クリップボード操作

**メソッド**:

```typescript
class ClipboardService {
  // テキストをクリップボードにコピー
  async copyToClipboard(text: string): Promise<void> {
    try {
      await navigator.clipboard.writeText(text);
    } catch (error) {
      // フォールバック: 古いブラウザ対応
      this.fallbackCopy(text);
    }
  }
  
  // フォールバック方式でのコピー
  private fallbackCopy(text: string): void {
    const textArea = document.createElement('textarea');
    textArea.value = text;
    textArea.style.position = 'fixed';
    textArea.style.left = '-999999px';
    document.body.appendChild(textArea);
    textArea.select();
    document.execCommand('copy');
    document.body.removeChild(textArea);
  }
}
```

---

## バックエンドサービス

### 4. BedrockService
**責任範囲**: Amazon Bedrockとの統合、プロンプト実行

**メソッド**:

```typescript
import { BedrockRuntimeClient, InvokeModelCommand } from '@aws-sdk/client-bedrock-runtime';

class BedrockService {
  private client: BedrockRuntimeClient;
  private modelId: string;
  
  constructor(region: string, modelId: string) {
    this.client = new BedrockRuntimeClient({ region });
    this.modelId = modelId;
  }
  
  // プロフィール分析の実行
  async analyzeProfile(
    phase1: Phase1Data,
    phase2: Phase2Data,
    promptTemplate: string
  ): Promise<AnalysisResult> {
    // プロンプトの構築
    const prompt = this.buildPrompt(phase1, phase2, promptTemplate);
    
    // Bedrock APIの呼び出し
    const response = await this.invokeBedrock(prompt);
    
    // レスポンスのパース
    return this.parseResponse(response);
  }
  
  // プロンプトの構築
  private buildPrompt(
    phase1: Phase1Data,
    phase2: Phase2Data,
    template: string
  ): string {
    return template
      .replace('{{selfIntro}}', phase1.selfIntro)
      .replace('{{hobbies}}', phase1.hobbies)
      .replace('{{idealPartner}}', phase1.idealPartner)
      .replace('{{weekendActivities}}', phase1.weekendActivities)
      .replace('{{lazyLevel}}', phase2.lazyLevel.toString())
      .replace('{{actualWeekend}}', phase2.actualWeekend)
      .replace('{{lazyHabits}}', phase2.lazyHabits)
      .replace('{{troublesome}}', phase2.troublesome);
  }
  
  // Bedrock APIの呼び出し
  private async invokeBedrock(prompt: string): Promise<string> {
    const command = new InvokeModelCommand({
      modelId: this.modelId,
      contentType: 'application/json',
      accept: 'application/json',
      body: JSON.stringify({
        anthropic_version: 'bedrock-2023-05-31',
        max_tokens: 4000,
        temperature: 0.7,
        messages: [
          {
            role: 'user',
            content: prompt,
          },
        ],
      }),
    });
    
    const response = await this.client.send(command);
    const responseBody = JSON.parse(new TextDecoder().decode(response.body));
    
    return responseBody.content[0].text;
  }
  
  // レスポンスのパース
  private parseResponse(response: string): AnalysisResult {
    // JSONレスポンスをパース
    const parsed = JSON.parse(response);
    
    return {
      phase3: {
        contradictions: parsed.phase3.contradictions,
        verdict: parsed.phase3.verdict,
      },
      phase4: {
        flawToCharmTranslation: parsed.phase4.flawToCharmTranslation,
        newTarget: parsed.phase4.newTarget,
        newProfile: parsed.phase4.newProfile,
        expectedMatchRate: parsed.phase4.expectedMatchRate,
      },
    };
  }
}
```

### 5. PromptService
**責任範囲**: プロンプトテンプレートの管理

**メソッド**:

```typescript
import { SSMClient, GetParameterCommand } from '@aws-sdk/client-ssm';

class PromptService {
  private ssmClient: SSMClient;
  private parameterName: string;
  
  constructor(region: string, parameterName: string) {
    this.ssmClient = new SSMClient({ region });
    this.parameterName = parameterName;
  }
  
  // プロンプトテンプレートの取得
  async getPromptTemplate(): Promise<string> {
    const command = new GetParameterCommand({
      Name: this.parameterName,
      WithDecryption: false,
    });
    
    const response = await this.ssmClient.send(command);
    
    if (!response.Parameter?.Value) {
      throw new Error('Prompt template not found in Parameter Store');
    }
    
    return response.Parameter.Value;
  }
}
```

### 6. ValidationService（バックエンド）
**責任範囲**: バックエンド側の入力検証

**メソッド**:

```typescript
class BackendValidationService {
  // リクエスト全体の検証
  validateRequest(request: AnalyzeRequest): void {
    this.validatePhase1Data(request.phase1);
    this.validatePhase2Data(request.phase2);
  }
  
  // Phase 1データの検証
  private validatePhase1Data(data: Phase1Data): void {
    if (!data.selfIntro || typeof data.selfIntro !== 'string') {
      throw new Error('Invalid selfIntro');
    }
    if (!data.hobbies || typeof data.hobbies !== 'string') {
      throw new Error('Invalid hobbies');
    }
    if (!data.idealPartner || typeof data.idealPartner !== 'string') {
      throw new Error('Invalid idealPartner');
    }
    if (!data.weekendActivities || typeof data.weekendActivities !== 'string') {
      throw new Error('Invalid weekendActivities');
    }
  }
  
  // Phase 2データの検証
  private validatePhase2Data(data: Phase2Data): void {
    if (typeof data.lazyLevel !== 'number' || data.lazyLevel < 1 || data.lazyLevel > 5) {
      throw new Error('Invalid lazyLevel');
    }
    if (!data.actualWeekend || typeof data.actualWeekend !== 'string') {
      throw new Error('Invalid actualWeekend');
    }
    if (!data.lazyHabits || typeof data.lazyHabits !== 'string') {
      throw new Error('Invalid lazyHabits');
    }
    if (!data.troublesome || typeof data.troublesome !== 'string') {
      throw new Error('Invalid troublesome');
    }
  }
}
```

### 7. ErrorService
**責任範囲**: エラーハンドリングとログ記録

**メソッド**:

```typescript
class ErrorService {
  // エラーのログ記録
  logError(error: Error, context: Record<string, any>): void {
    console.error('Error occurred:', {
      message: error.message,
      stack: error.stack,
      context,
      timestamp: new Date().toISOString(),
    });
  }
  
  // エラーレスポンスの生成
  createErrorResponse(error: Error): APIGatewayProxyResult {
    const errorType = this.determineErrorType(error);
    const statusCode = this.getStatusCode(errorType);
    const userMessage = this.getUserMessage(errorType);
    
    return {
      statusCode,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
      },
      body: JSON.stringify({
        error: userMessage,
      }),
    };
  }
  
  // エラー種別の判定
  private determineErrorType(error: Error): ErrorType {
    if (error.message.includes('Invalid')) {
      return ErrorType.VALIDATION_ERROR;
    }
    if (error.message.includes('timeout')) {
      return ErrorType.TIMEOUT_ERROR;
    }
    if (error.message.includes('Bedrock')) {
      return ErrorType.BEDROCK_ERROR;
    }
    return ErrorType.INTERNAL_ERROR;
  }
  
  // ステータスコードの取得
  private getStatusCode(errorType: ErrorType): number {
    switch (errorType) {
      case ErrorType.VALIDATION_ERROR:
        return 400;
      case ErrorType.TIMEOUT_ERROR:
        return 504;
      case ErrorType.BEDROCK_ERROR:
        return 502;
      default:
        return 500;
    }
  }
  
  // ユーザー向けメッセージの生成
  private getUserMessage(errorType: ErrorType): string {
    switch (errorType) {
      case ErrorType.VALIDATION_ERROR:
        return '入力データが不正です';
      case ErrorType.TIMEOUT_ERROR:
        return 'サーバーの応答がタイムアウトしました';
      case ErrorType.BEDROCK_ERROR:
        return 'AI分析サービスでエラーが発生しました';
      default:
        return '予期しないエラーが発生しました';
    }
  }
}
```

---

## オーケストレーション

### Lambda Handler のオーケストレーション

```typescript
// Lambda関数のメインハンドラー
export const handler = async (
  event: APIGatewayProxyEvent,
  context: Context
): Promise<APIGatewayProxyResult> => {
  // サービスのインスタンス化
  const validationService = new BackendValidationService();
  const promptService = new PromptService(
    process.env.AWS_REGION!,
    process.env.PROMPT_PARAMETER_NAME!
  );
  const bedrockService = new BedrockService(
    process.env.AWS_REGION!,
    process.env.BEDROCK_MODEL_ID!
  );
  const errorService = new ErrorService();
  
  try {
    // 1. リクエストボディのパース
    const request: AnalyzeRequest = JSON.parse(event.body || '{}');
    
    // 2. バリデーション
    validationService.validateRequest(request);
    
    // 3. プロンプトテンプレートの取得
    const promptTemplate = await promptService.getPromptTemplate();
    
    // 4. Bedrockによる分析実行
    const result = await bedrockService.analyzeProfile(
      request.phase1,
      request.phase2,
      promptTemplate
    );
    
    // 5. 成功レスポンスの返却
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
      },
      body: JSON.stringify(result),
    };
  } catch (error) {
    // エラーハンドリング
    errorService.logError(error as Error, {
      requestId: context.requestId,
      functionName: context.functionName,
    });
    
    return errorService.createErrorResponse(error as Error);
  }
};
```

### フロントエンドのオーケストレーション

```typescript
// Appコンポーネントでのオーケストレーション
const App: React.FC = () => {
  const [state, setState] = useState<AppState>({
    currentPhase: 1,
    phase1Data: null,
    phase2Data: null,
    analysisResult: null,
    isLoading: false,
    error: null,
  });
  
  // サービスのインスタンス化
  const apiService = new ApiService(process.env.NEXT_PUBLIC_API_URL!);
  const validationService = new ValidationService();
  
  // Phase 2データ保存と分析実行
  const savePhase2DataAndAnalyze = async (data: Phase2Data) => {
    try {
      // 1. バリデーション
      const validation = validationService.validatePhase2(data);
      if (!validation.isValid) {
        setState(prev => ({ ...prev, error: validation.errors.join(', ') }));
        return;
      }
      
      // 2. ローディング開始
      setState(prev => ({ ...prev, isLoading: true, error: null }));
      
      // 3. API呼び出し
      const result = await apiService.analyzeProfile(
        state.phase1Data!,
        data
      );
      
      // 4. 結果の保存とPhase 3への遷移
      setState(prev => ({
        ...prev,
        phase2Data: data,
        analysisResult: result,
        currentPhase: 3,
        isLoading: false,
      }));
    } catch (error) {
      // エラーハンドリング
      const errorMessage = apiService.handleApiError(error as Error);
      setState(prev => ({ ...prev, error: errorMessage, isLoading: false }));
    }
  };
  
  // レンダリング
  return (
    <div>
      {state.currentPhase === 1 && <Phase1Page onNext={savePhase1Data} />}
      {state.currentPhase === 2 && <Phase2Page onSubmit={savePhase2DataAndAnalyze} />}
      {state.currentPhase === 3 && <Phase3Page result={state.analysisResult!.phase3} />}
      {state.currentPhase === 4 && <Phase4Page result={state.analysisResult!.phase4} />}
      {state.isLoading && <LoadingSpinner />}
      {state.error && <ErrorMessage message={state.error} />}
    </div>
  );
};
```
