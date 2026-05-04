# コンポーネント定義 - ダメ恋プロフィール裁判所

## フロントエンドコンポーネント

### 1. App（ルートコンポーネント）
**責任範囲**:
- アプリケーション全体の状態管理
- Phase遷移の制御
- API通信の統括

**主要な状態**:
- currentPhase: 現在のPhase（1-4）
- phase1Data: Phase 1の入力データ
- phase2Data: Phase 2の入力データ
- analysisResult: バックエンドからの分析結果
- isLoading: ローディング状態
- error: エラー情報

### 2. Phase1Page（虚飾の入力）
**責任範囲**:
- Phase 1の入力フォーム表示
- 入力データの検証
- Phase 2への遷移

**入力項目**:
- 自己紹介文
- 趣味・特技
- 理想の相手像
- 休日の過ごし方

### 3. Phase2Page（本音の入力）
**責任範囲**:
- Phase 2の入力フォーム表示
- 入力データの検証
- バックエンドAPIへのデータ送信

**入力項目**:
- ズボラ度（1-5スライダー）
- 実際の休日の過ごし方
- サボり癖・怠惰な習慣
- 本当は面倒だと思っていること

### 4. Phase3Page（論理的否定）
**責任範囲**:
- 論理的否定の結果表示
- 暗転演出の制御
- Phase 4への遷移

**表示内容**:
- 矛盾の指摘
- 学術的根拠に基づく戦略破綻の証明
- 判決文

### 5. Phase4Page（改善案発行）
**責任範囲**:
- 改善案の結果表示
- 抹消線アニメーションの制御
- 新プロフィール文のコピー機能

**表示内容**:
- ダメさを魅力に翻訳した説明
- 新ターゲット層の定義
- 新プロフィール文
- 期待マッチング率

### 6. InputField（共通UIコンポーネント）
**責任範囲**:
- テキスト入力フィールドの表示
- バリデーション表示

**Props**:
- label: ラベルテキスト
- value: 入力値
- onChange: 変更ハンドラ
- placeholder: プレースホルダー
- error: エラーメッセージ

### 7. Slider（共通UIコンポーネント）
**責任範囲**:
- スライダー入力の表示
- 値の視覚的フィードバック

**Props**:
- label: ラベルテキスト
- value: 現在の値（1-5）
- onChange: 変更ハンドラ
- min: 最小値
- max: 最大値

### 8. Button（共通UIコンポーネント）
**責任範囲**:
- ボタンの表示
- クリックイベントの処理

**Props**:
- text: ボタンテキスト
- onClick: クリックハンドラ
- disabled: 無効化フラグ
- variant: スタイルバリアント（primary/secondary）

### 9. LoadingSpinner（共通UIコンポーネント）
**責任範囲**:
- ローディング表示
- ローディングメッセージの表示

**Props**:
- message: ローディングメッセージ

### 10. ErrorMessage（共通UIコンポーネント）
**責任範囲**:
- エラーメッセージの表示
- エラーの視覚的フィードバック

**Props**:
- message: エラーメッセージ
- onClose: 閉じるハンドラ

### 11. CopyButton（共通UIコンポーネント）
**責任範囲**:
- クリップボードへのコピー
- コピー完了フィードバック

**Props**:
- text: コピーするテキスト
- onCopySuccess: コピー成功時のコールバック

### 12. DarkenTransition（アニメーションコンポーネント）
**責任範囲**:
- 画面暗転演出
- テキスト浮かび上がり演出

**Props**:
- isActive: 演出の有効化フラグ
- children: 表示するコンテンツ
- duration: 演出時間（ミリ秒）

### 13. StrikethroughAnimation（アニメーションコンポーネント）
**責任範囲**:
- 抹消線アニメーション
- テキスト上書き演出

**Props**:
- oldText: 元のテキスト
- newText: 新しいテキスト
- isActive: 演出の有効化フラグ

---

## バックエンドコンポーネント

### 14. AnalyzeHandler（Lambda関数）
**責任範囲**:
- API Gatewayからのリクエスト受信
- リクエストデータの検証
- Bedrockサービスの呼び出し
- レスポンスの整形と返却
- エラーハンドリング

**入力**:
- phase1Data: Phase 1の入力データ
- phase2Data: Phase 2の入力データ

**出力**:
- phase3Result: 論理的否定の結果
- phase4Result: 改善案の結果

### 15. BedrockService（サービスクラス）
**責任範囲**:
- Bedrock APIの呼び出し
- プロンプトの構築
- レスポンスのパース
- タイムアウト処理

**メソッド**:
- analyzeProfile(): プロフィール分析の実行
- buildPrompt(): プロンプトの構築
- parseResponse(): レスポンスのパース

### 16. PromptManager（ユーティリティクラス）
**責任範囲**:
- Parameter Storeからのプロンプト取得
- プロンプトテンプレートの管理
- プロンプトへのデータ埋め込み

**メソッド**:
- getPromptTemplate(): プロンプトテンプレート取得
- fillPromptData(): データ埋め込み

### 17. ValidationService（サービスクラス）
**責任範囲**:
- 入力データの検証
- データ型チェック
- 必須項目チェック

**メソッド**:
- validatePhase1Data(): Phase 1データの検証
- validatePhase2Data(): Phase 2データの検証

### 18. ErrorHandler（ユーティリティクラス）
**責任範囲**:
- エラーのログ記録
- エラーレスポンスの生成
- エラー種別の判定

**メソッド**:
- logError(): CloudWatch Logsへのエラー記録
- createErrorResponse(): エラーレスポンス生成

---

## インフラコンポーネント

### 19. DameKoiStack（CDKスタック）
**責任範囲**:
- 全AWSリソースの定義
- リソース間の接続設定
- 環境変数の設定

**リソース**:
- API Gateway（REST API）
- Lambda関数
- IAMロール・ポリシー
- Parameter Store（プロンプト保存用）
- CloudWatch Logs

### 20. ApiGateway（CDKコンストラクト）
**責任範囲**:
- REST APIの定義
- エンドポイントの設定
- CORSの設定
- レート制限の設定

**エンドポイント**:
- POST /api/analyze

### 21. LambdaFunction（CDKコンストラクト）
**責任範囲**:
- Lambda関数の定義
- 環境変数の設定
- タイムアウト設定
- メモリ設定

**設定**:
- Runtime: Node.js 18.x または Python 3.11
- Timeout: 30秒
- Memory: 512MB

### 22. BedrockPermissions（CDKコンストラクト）
**責任範囲**:
- Bedrock APIへのアクセス権限設定
- IAMポリシーの定義

**権限**:
- bedrock:InvokeModel

### 23. ParameterStoreConfig（CDKコンストラクト）
**責任範囲**:
- Parameter Storeの設定
- プロンプトテンプレートの保存

**パラメータ**:
- /damekoi/prompt/template
