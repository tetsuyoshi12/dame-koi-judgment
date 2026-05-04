# Unit of Work Story Map - ダメ恋プロフィール裁判所

## ストーリーマッピング概要

本ドキュメントでは、要件定義書（requirements.md）で定義された機能要件を3つのユニット（Frontend、Backend、Infrastructure）にマッピングします。

---

## ユニット別ストーリーマッピング

### Frontend Unit

#### Phase 1: 虚飾の入力

| 要件ID | 要件内容 | 実装コンポーネント |
|--------|----------|-------------------|
| REQ-F1.1 | マッチングアプリ風の入力フォームを表示 | Phase1Page |
| REQ-F1.2 | 自己紹介文の入力受付（テキストエリア） | InputField |
| REQ-F1.3 | 趣味・特技の入力受付（テキストエリア） | InputField |
| REQ-F1.4 | 理想の相手像の入力受付（テキストエリア） | InputField |
| REQ-F1.5 | 休日の過ごし方の入力受付（テキストエリア） | InputField |
| REQ-F1.6 | Phase 2画面への遷移 | App（状態管理） |

#### Phase 2: 本音の入力

| 要件ID | 要件内容 | 実装コンポーネント |
|--------|----------|-------------------|
| REQ-F2.1 | 本音入力フォームを表示 | Phase2Page |
| REQ-F2.2 | ズボラ度の入力受付（1-5スライダー） | Slider |
| REQ-F2.3 | 実際の休日の過ごし方の入力受付（テキストエリア） | InputField |
| REQ-F2.4 | サボり癖・怠惰な習慣の入力受付（テキストエリア） | InputField |
| REQ-F2.5 | 本当は面倒だと思っていることの入力受付（テキストエリア） | InputField |
| REQ-F2.6 | バックエンドAPIへのデータ送信 | ApiService |
| REQ-F2.7 | ローディング表示 | LoadingSpinner |

#### Phase 3: 論理的否定（結果表示）

| 要件ID | 要件内容 | 実装コンポーネント |
|--------|----------|-------------------|
| REQ-F3.1 | 画面の暗転 | DarkenTransition |
| REQ-F3.2 | 2〜3秒間の無音状態維持 | DarkenTransition |
| REQ-F3.3 | 論理的否定文の浮かび上がり表示 | Phase3Page |
| REQ-F3.4 | AIによる矛盾の指摘を表示 | Phase3Page |
| REQ-F3.5 | 学術的根拠に基づく戦略破綻の証明を表示 | Phase3Page |
| REQ-F3.6 | 「頑張っても無駄」の宣告を表示 | Phase3Page |
| REQ-F3.7 | 冷徹な裁判官トーンで表示 | Phase3Page |
| REQ-F3.8 | Phase 4画面への遷移ボタン表示 | Button |

#### Phase 4: 改善案発行（結果表示）

| 要件ID | 要件内容 | 実装コンポーネント |
|--------|----------|-------------------|
| REQ-F4.1 | 改善案を表示 | Phase4Page |
| REQ-F4.2 | ダメさを魅力に翻訳した説明を表示 | Phase4Page |
| REQ-F4.3 | 新ターゲット層の定義を表示 | Phase4Page |
| REQ-F4.4 | 150〜300文字の新プロフィール文を表示 | Phase4Page |
| REQ-F4.5 | 期待マッチング率を表示 | Phase4Page |
| REQ-F4.6 | Phase 1の入力文に抹消線を表示 | StrikethroughAnimation |
| REQ-F4.7 | CSSアニメーションで視覚的な転換を演出 | StrikethroughAnimation |

#### コピー機能

| 要件ID | 要件内容 | 実装コンポーネント |
|--------|----------|-------------------|
| REQ-F7.1 | 新プロフィール文をクリップボードにコピー | ClipboardService, CopyButton |
| REQ-F7.2 | コピー完了のフィードバックメッセージ表示 | CopyButton |
| REQ-F7.3 | フィードバックメッセージの自動非表示（2〜3秒後） | CopyButton |

#### UI/UX演出

| 要件ID | 要件内容 | 実装コンポーネント |
|--------|----------|-------------------|
| REQ-F6.1 | Phase 1の入力文に抹消線を表示 | StrikethroughAnimation |
| REQ-F6.2 | CSSアニメーションで視覚的な転換を演出 | StrikethroughAnimation |
| REQ-F6.3 | 画面の暗転 | DarkenTransition |
| REQ-F6.4 | 2〜3秒間待機 | DarkenTransition |
| REQ-F6.5 | 結果テキストの浮かび上がり表示 | Phase3Page |
| REQ-F6.6 | 冷徹な裁判官トーンの文体使用 | Phase3Page |
| REQ-F6.7 | 判決文形式で表示 | Phase3Page |

---

### Backend Unit

#### Amazon Bedrock統合

| 要件ID | 要件内容 | 実装コンポーネント |
|--------|----------|-------------------|
| REQ-F5.1 | Amazon Bedrock APIの呼び出し | BedrockService |
| REQ-F5.2 | Claude Opus 4.7（または最新Claude）の使用 | BedrockService |
| REQ-F5.3 | Phase 1とPhase 2の矛盾検知指示 | PromptService |
| REQ-F5.4 | 学術的根拠付き論理的否定文の生成指示 | PromptService |
| REQ-F5.5 | 行動心理学・統計データ・進化学的視点の用語を含める | PromptService |
| REQ-F5.6 | 事実の指摘・学術的根拠・甘い期待の棄却の3要素を含める | PromptService |
| REQ-F5.7 | ダメさを魅力に翻訳する指示 | PromptService |
| REQ-F5.8 | 新プロフィール文の発行指示 | PromptService |
| REQ-F5.9 | 期待マッチング率の生成指示 | PromptService |
| REQ-F5.10 | 15秒以内にフロントエンドに結果を返す | AnalyzeHandler |
| REQ-F5.11 | 15秒超過時のタイムアウトエラー返却 | ErrorService |

#### バリデーションとエラーハンドリング

| 機能 | 実装コンポーネント |
|------|-------------------|
| 入力データのバリデーション | BackendValidationService |
| エラーハンドリングとログ記録 | ErrorService |
| CloudWatch Logsへのログ出力 | Logger（utils） |

---

### Infrastructure Unit

#### AWSリソース管理

| リソース | 実装コンポーネント |
|---------|-------------------|
| API Gateway設定 | ApiGatewayConstruct |
| Lambda関数設定 | LambdaConstruct |
| IAMロール・ポリシー設定 | BedrockPermissionsConstruct |
| Parameter Store設定 | ParameterStoreConstruct |
| CloudWatch Logs設定 | DameKoiStack |

#### デプロイ管理

| 機能 | 実装コンポーネント |
|------|-------------------|
| CDKスタックの定義 | DameKoiStack |
| デプロイスクリプト | deploy.sh |
| リソース削除スクリプト | destroy.sh |
| プロンプトテンプレート管理 | prompt-template.txt |

---

## ユニット間のストーリーフロー

### エンドツーエンドフロー

```
[ユーザー]
    |
    | Phase 1入力（REQ-F1.1〜REQ-F1.6）
    v
[Frontend: Phase1Page]
    |
    | Phase 2入力（REQ-F2.1〜REQ-F2.7）
    v
[Frontend: Phase2Page]
    |
    | API呼び出し（REQ-F2.6）
    v
[Frontend: ApiService]
    |
    | HTTP POST /api/analyze
    v
[Infrastructure: API Gateway]
    |
    | Lambda トリガー
    v
[Backend: AnalyzeHandler]
    |
    | バリデーション
    v
[Backend: ValidationService]
    |
    | プロンプト取得
    v
[Backend: PromptService] → [Infrastructure: Parameter Store]
    |
    | AI分析（REQ-F5.1〜REQ-F5.11）
    v
[Backend: BedrockService] → [AWS: Bedrock]
    |
    | レスポンス生成
    v
[Backend: AnalyzeHandler]
    |
    | HTTP Response
    v
[Frontend: ApiService]
    |
    | Phase 3表示（REQ-F3.1〜REQ-F3.8）
    v
[Frontend: Phase3Page + DarkenTransition]
    |
    | Phase 4表示（REQ-F4.1〜REQ-F4.7）
    v
[Frontend: Phase4Page + StrikethroughAnimation]
    |
    | コピー機能（REQ-F7.1〜REQ-F7.3）
    v
[Frontend: CopyButton + ClipboardService]
    |
    v
[ユーザー]
```

---

## ユニット別開発タスクマッピング

### Frontend Unit 開発タスク

| タスク | 関連ストーリー | 優先度 |
|--------|--------------|--------|
| プロジェクトセットアップ（Next.js初期化） | - | 1 |
| 型定義の作成（shared/から参照） | - | 1 |
| 共通UIコンポーネントの実装 | REQ-F1.2〜F1.5, REQ-F2.2〜F2.5, REQ-F3.8 | 1 |
| Phase 1, 2の入力ページ実装 | REQ-F1.1〜F1.6, REQ-F2.1〜F2.7 | 1 |
| ApiServiceの実装 | REQ-F2.6, REQ-F2.7 | 2 |
| Phase 3, 4の結果表示ページ実装 | REQ-F3.1〜F3.8, REQ-F4.1〜F4.7 | 4 |
| アニメーションコンポーネントの実装 | REQ-F3.1〜F3.3, REQ-F4.6〜F4.7, REQ-F6.1〜F6.7 | 6 |
| コピー機能の実装 | REQ-F7.1〜F7.3 | 7 |
| 単体テストの作成 | - | 8 |
| 統合テスト（手動） | - | 8 |

### Backend Unit 開発タスク

| タスク | 関連ストーリー | 優先度 |
|--------|--------------|--------|
| プロジェクトセットアップ（TypeScript + AWS SDK） | - | 2 |
| 型定義の作成（shared/から参照） | - | 2 |
| ValidationServiceの実装 | - | 2 |
| PromptServiceの実装（Parameter Store連携） | REQ-F5.3〜F5.9 | 3 |
| BedrockServiceの実装（Bedrock API連携） | REQ-F5.1〜F5.2, REQ-F5.10〜F5.11 | 2 |
| ErrorServiceの実装 | REQ-F5.11 | 2 |
| AnalyzeHandlerの実装（オーケストレーション） | REQ-F5.10 | 2 |
| 単体テストの作成 | - | 8 |
| ローカルテスト（SAM Local または手動テスト） | - | 8 |

### Infrastructure Unit 開発タスク

| タスク | 関連ストーリー | 優先度 |
|--------|--------------|--------|
| CDKプロジェクトセットアップ | - | 5 |
| DameKoiStackの実装 | - | 5 |
| API Gateway設定 | - | 5 |
| Lambda関数設定（Backend Unitのコードを参照） | - | 5 |
| IAMロール・ポリシー設定 | - | 5 |
| Parameter Store設定（プロンプトテンプレート保存） | REQ-F5.3〜F5.9 | 5 |
| CloudWatch Logs設定 | - | 5 |
| デプロイスクリプトの作成 | - | 5 |
| デプロイテスト | - | 5 |

---

## ストーリー完成の定義

### Frontend Unit ストーリー完成条件

- [ ] REQ-F1.1〜F1.6: Phase 1入力フォームが実装され、Phase 2への遷移が動作する
- [ ] REQ-F2.1〜F2.7: Phase 2入力フォームが実装され、API呼び出しとローディング表示が動作する
- [ ] REQ-F3.1〜F3.8: Phase 3結果表示が実装され、暗転演出と論理的否定文の表示が動作する
- [ ] REQ-F4.1〜F4.7: Phase 4改善案表示が実装され、抹消線アニメーションが動作する
- [ ] REQ-F6.1〜F6.7: UI/UX演出がすべて実装され、視覚的な転換が動作する
- [ ] REQ-F7.1〜F7.3: コピー機能が実装され、クリップボード操作とフィードバック表示が動作する
- [ ] 主要機能の単体テストがパスする

### Backend Unit ストーリー完成条件

- [ ] REQ-F5.1〜F5.2: Bedrock APIとの統合が完了し、Claude Opus 4.7が使用できる
- [ ] REQ-F5.3〜F5.9: プロンプトサービスが実装され、すべての生成指示が正しく動作する
- [ ] REQ-F5.10: 15秒以内にフロントエンドに結果を返すことができる
- [ ] REQ-F5.11: タイムアウトエラーハンドリングが正しく動作する
- [ ] バリデーションとエラーハンドリングが適切に実装されている
- [ ] 主要機能の単体テストがパスする

### Infrastructure Unit ストーリー完成条件

- [ ] CDKスタックがデプロイできる
- [ ] API Gateway が正常に動作する
- [ ] Lambda関数が正常にトリガーされる
- [ ] IAM権限が適切に設定されている
- [ ] Parameter Store にプロンプトが保存されている
- [ ] CloudWatch Logs にログが記録される

---

## ストーリーポイント推定（参考）

### Frontend Unit

| ストーリーグループ | ストーリーポイント | 理由 |
|------------------|-------------------|------|
| Phase 1, 2入力フォーム | 5 | 標準的なフォーム実装 |
| Phase 3, 4結果表示 | 5 | データ表示とレイアウト |
| アニメーション演出 | 8 | CSS Animationsの実装と調整 |
| API統合 | 3 | fetch APIの実装 |
| コピー機能 | 2 | Clipboard APIの実装 |
| 単体テスト | 3 | 主要機能のテスト |
| **合計** | **26** | |

### Backend Unit

| ストーリーグループ | ストーリーポイント | 理由 |
|------------------|-------------------|------|
| Bedrock統合 | 8 | AWS SDK統合とエラーハンドリング |
| プロンプトサービス | 5 | Parameter Store連携とプロンプト生成 |
| バリデーション | 3 | 入力検証ロジック |
| エラーハンドリング | 3 | エラーサービスとログ記録 |
| 単体テスト | 3 | 主要機能のテスト |
| **合計** | **22** | |

### Infrastructure Unit

| ストーリーグループ | ストーリーポイント | 理由 |
|------------------|-------------------|------|
| CDKスタック実装 | 8 | すべてのAWSリソース定義 |
| デプロイスクリプト | 2 | デプロイ自動化 |
| デプロイテスト | 3 | 統合テストと検証 |
| **合計** | **13** | |

### プロジェクト全体

| ユニット | ストーリーポイント |
|---------|-------------------|
| Frontend | 26 |
| Backend | 22 |
| Infrastructure | 13 |
| **合計** | **61** |

**推定開発期間**: 2週間（2-3人チーム）

---

## リスクとストーリーへの影響

### リスク1: Bedrock（Claude Opus 4.7）が利用可能なリージョンが限定的
- **影響ストーリー**: REQ-F5.1〜F5.2
- **対策**: 事前にリージョン可用性を確認、代替モデル（Claude Sonnet）も検討

### リスク2: プロンプトがAI Safetyフィルターに引っかかる
- **影響ストーリー**: REQ-F5.3〜F5.9
- **対策**: 学術用語・統計用語で攻撃性を客観的事実報告に偽装

### リスク3: Bedrockのレスポンスが15秒を超える
- **影響ストーリー**: REQ-F5.10〜F5.11
- **対策**: プロンプトを最適化してトークン消費を削減、ローディング表示で体感時間を短縮

### リスク4: 2週間で実装が完了しない
- **影響ストーリー**: すべて
- **対策**: MVP（Minimum Viable Product）を明確化、演出は最後に実装

---

## まとめ

本ストーリーマップにより、以下が明確になりました：

1. **Frontend Unit**: 33個の機能要件のうち、26個を担当（Phase 1〜4のUI、演出、コピー機能）
2. **Backend Unit**: 11個の機能要件を担当（Bedrock統合、プロンプト生成、バリデーション）
3. **Infrastructure Unit**: AWSリソース管理とデプロイを担当（機能要件には直接対応しないが、すべての要件の実現基盤）

各ユニットは独立して開発可能であり、OpenAPI仕様書を契約として並行開発が可能です。
