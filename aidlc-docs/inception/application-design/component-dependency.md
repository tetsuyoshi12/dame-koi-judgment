# コンポーネント依存関係

## 依存関係マトリクス

| コンポーネント | 依存先 | 依存タイプ | 理由 |
|--------------|--------|-----------|------|
| App | SessionOrchestrationService (Redux Store) | 直接 | Store プロバイダーとして初期化 |
| Phase1Page | SessionOrchestrationService | 直接 | phase1Data の保存・currentPhase の更新 |
| Phase2Page | SessionOrchestrationService | 直接 | phase2Questions/Answers の保存 |
| Phase2Page | ApiClientService | 直接 | `/generate-questions` 呼び出し |
| Phase3Page | SessionOrchestrationService | 直接 | analysisResult の保存 |
| Phase3Page | ApiClientService | 直接 | `/analyze` 呼び出し |
| Phase3Page | AnimationOrchestrationService | 直接 | 暗転・浮上アニメーション制御 |
| Phase4Page | SessionOrchestrationService | 直接 | analysisResult の読み取り |
| Phase4Page | AnimationOrchestrationService | 直接 | 抹消線・上書きアニメーション制御 |
| MonolithCharacter | SessionOrchestrationService | 直接 | currentPhase に応じた状態表示 |
| GenerateQuestionsLambda | BedrockPromptService | 直接 | プロンプト構築・Bedrock 呼び出し |
| AnalyzeLambda | BedrockPromptService | 直接 | プロンプト構築・Bedrock 呼び出し |
| AnalyzeLambda | SessionStorageService | 直接 | セッションデータの S3 保存 |
| BedrockPromptService | INFRA-02 (S3 データバケット) | 外部 | プロンプトテンプレート読み込み |
| SessionStorageService | INFRA-02 (S3 データバケット) | 外部 | セッションデータ保存 |
| GenerateQuestionsLambda | INFRA-04 (Bedrock) | 外部 | AI 推論 |
| AnalyzeLambda | INFRA-04 (Bedrock) | 外部 | AI 推論 |
| ApiClientService | INFRA-01 (API Gateway) | 外部 | HTTP 通信 |

---

## データフロー図

```
[ユーザー]
    |
    | 虚飾プロフィール入力
    v
[Phase1Page] ---> [SessionOrchestrationService (Redux)]
                          |
                          | phase1Data
                          v
[Phase2Page] ---> [ApiClientService] ---> [API Gateway]
                                               |
                                               | POST /generate-questions
                                               v
                                    [GenerateQuestionsLambda]
                                               |
                                    [BedrockPromptService]
                                               |
                                    [S3: prompts/]  [Bedrock Claude Opus 4.7]
                                               |
                          質問リスト           |
                          <-------------------+
                          |
[Phase2Page] <--- 本音質問表示
    |
    | 本音回答入力
    v
[SessionOrchestrationService (Redux)]
    |
    | phase1Data + phase2Answers
    v
[Phase3Page] ---> [ApiClientService] ---> [API Gateway]
                                               |
                                               | POST /analyze
                                               v
                                    [AnalyzeLambda]
                                               |
                                    [BedrockPromptService]
                                               |
                              [S3: prompts/]  [Bedrock Claude Opus 4.7]
                                               |
                                    [SessionStorageService]
                                               |
                                    [S3: sessions/]
                                               |
                          分析結果             |
                          <-------------------+
                          |
[Phase3Page] ---> [AnimationOrchestrationService]
    |              (暗転 -> 否定文浮上)
    |
    v
[Phase4Page] ---> [AnimationOrchestrationService]
                   (抹消線 -> 上書き)
```

---

## 通信パターン

### フロントエンド ↔ バックエンド
- **プロトコル**: HTTPS（REST）
- **認証**: なし（ハッカソン用途）
- **タイムアウト**: `/generate-questions` 5秒、`/analyze` 20秒（Lambda タイムアウト + バッファ）
- **エラーレスポンス**: HTTP 500 + エラーメッセージ JSON

### バックエンド ↔ AWS サービス
- **Bedrock**: AWS SDK（boto3）、`invoke_model` API
- **S3**: AWS SDK（boto3）、`get_object` / `put_object`
- **認証**: Lambda 実行ロールの IAM ポリシー
