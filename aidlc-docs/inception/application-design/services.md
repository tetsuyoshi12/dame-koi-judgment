# サービス定義

## SVC-01: SessionOrchestrationService（フロントエンド）

**目的**: 4フェーズをまたぐセッションデータの一元管理とフェーズ遷移の制御

**責務**:
- Redux Store を通じた全フェーズデータの保持
  - `phase1Data`: 虚飾プロフィール入力値
  - `phase2Questions`: Bedrock が生成した本音質問リスト
  - `phase2Answers`: ユーザーの本音回答
  - `analysisResult`: Phase 3/4 の分析結果（否定文・翻訳テーブル・新プロフィール・マッチング率）
  - `currentPhase`: 現在のフェーズ番号（1〜4）
  - `sessionId`: セッション識別子（フロントエンドで生成）
- フェーズ遷移ロジック（前フェーズのデータが揃っていることを確認してから遷移）
- API 呼び出し状態管理（loading / success / error）

**インタラクション**:
- Phase1Page → `setPhase1Data()` → Redux Store
- Phase2Page → `setPhase2Questions()` / `setPhase2Answers()` → Redux Store
- Phase3Page / Phase4Page → `setAnalysisResult()` → Redux Store

---

## SVC-02: ApiClientService（フロントエンド）

**目的**: バックエンド API との通信を抽象化する

**責務**:
- `POST /generate-questions` の呼び出しと結果の型変換
- `POST /analyze` の呼び出しと結果の型変換
- エラーハンドリング（タイムアウト・ネットワークエラー・5xx エラー）
- リクエスト/レスポンスの型安全な変換（TypeScript 型定義）

**エンドポイント**:
```
POST /generate-questions
  Request:  { session_id: string, phase1_data: Phase1FormData }
  Response: { questions: Question[] }

POST /analyze
  Request:  { session_id: string, phase1_data: Phase1FormData, phase2_data: Phase2AnswerData }
  Response: { verdict: string, translation_table: TranslationItem[], new_profile: string, target_audience: string, match_rate_before: number, match_rate_after: number }
```

---

## SVC-03: AnimationOrchestrationService（フロントエンド）

**目的**: Phase 3/4 の演出アニメーションシーケンスを制御する

**責務**:
- Phase 3 暗転シーケンス: `fadeToBlack()` → `holdSilence(2500ms)` → `revealVerdict()`
- Phase 4 抹消線シーケンス: `applyStrikethrough()` → `revealNewText()`
- アニメーション完了コールバックの管理
- スキップ機能（アニメーション途中でスキップボタン押下時）
- Pure CSS クラスの動的付与・除去

---

## SVC-04: BedrockPromptService（バックエンド・共通）

**目的**: Bedrock へのプロンプト構築と呼び出しを管理する

**責務**:
- S3 からプロンプトテンプレートを読み込む（起動時キャッシュ）
- Phase 1 データを質問生成プロンプトに埋め込む
- Phase 1/2 対応ペアを分析プロンプトに埋め込む
- Bedrock API（`invoke_model`）の呼び出しとレスポンスのパース
- トークン消費の最適化（不要なコンテキストの除去）

**プロンプトテンプレート（S3 管理）**:
```
s3://{bucket}/prompts/generate-questions-template.json
s3://{bucket}/prompts/analyze-template.json
```

---

## SVC-05: SessionStorageService（バックエンド）

**目的**: 匿名化セッションデータの S3 への保存を管理する

**責務**:
- UUID v4 によるセッションID生成
- 個人識別情報を含まないことの確認
- S3 への JSON 保存（`s3://{bucket}/sessions/{session_id}.json`）
- 保存失敗時のエラーハンドリング（保存失敗はユーザー体験をブロックしない）

**保存データ構造**:
```json
{
  "session_id": "uuid-v4",
  "timestamp": "ISO8601",
  "phase1_data": { ... },
  "phase2_data": { ... },
  "analysis_result": { ... }
}
```
