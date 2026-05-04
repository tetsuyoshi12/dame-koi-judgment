# Application Design Plan - ダメ恋プロフィール裁判所

## 設計アプローチ

このプランでは、以下の成果物を生成します：
- [x] components.md - コンポーネント定義と責任範囲
- [x] component-methods.md - 各コンポーネントのメソッドシグネチャ
- [x] services.md - サービス定義とオーケストレーション
- [x] component-dependency.md - コンポーネント間の依存関係
- [x] application-design.md - 統合設計ドキュメント

## 設計質問

以下の質問に回答してください。各質問の後の [Answer]: の後に、選択肢の記号（A、B、C等）を記入してください。

### Question 1: フロントエンドのコンポーネント構成
4フェーズのUIをどのようにコンポーネント分割しますか？

A) ページ単位（Phase1Page、Phase2Page、Phase3Page、Phase4Page）
B) 機能単位（InputForm、ResultDisplay、Animation、CopyButton）
C) ハイブリッド（各PhasePageコンポーネント + 共通のUIコンポーネント）
D) その他（[Answer]: タグの後に説明してください）

[Answer]: C

### Question 2: 状態管理の方法
フロントエンドの状態管理はどのように実装しますか？

A) React Context API
B) Redux / Redux Toolkit
C) Zustand
D) useState + props（シンプルな状態管理）
E) その他（[Answer]: タグの後に説明してください）

[Answer]: D

### Question 3: バックエンドのLambda構成
バックエンドのLambda関数はどのように構成しますか？

A) 単一Lambda（/api/analyze エンドポイント1つ）
B) 複数Lambda（矛盾検知用、否定生成用、改善案生成用に分離）
C) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 4: Bedrockプロンプトの管理
Bedrockに送信するプロンプトはどこで管理しますか？

A) Lambda関数内にハードコード
B) 環境変数として管理
C) S3に保存してLambdaから読み込み
D) Parameter Storeに保存
E) その他（[Answer]: タグの後に説明してください）

[Answer]: D

### Question 5: エラーハンドリングの方針
エラーが発生した場合、どのように処理しますか？

A) ユーザーに汎用エラーメッセージを表示
B) エラーの種類に応じて具体的なメッセージを表示
C) エラーログをCloudWatch Logsに記録 + ユーザーに汎用メッセージ
D) その他（[Answer]: タグの後に説明してください）

[Answer]: C

### Question 6: API通信の設計
フロントエンドからバックエンドへのAPI通信はどのように実装しますか？

A) fetch API（ネイティブ）
B) axios
C) Next.jsのAPI Routes経由
D) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 7: レスポンスのキャッシング
Bedrockのレスポンスをキャッシュしますか？

A) キャッシュしない（毎回新規生成）
B) セッション単位でキャッシュ（同じ入力なら再利用）
C) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 8: フロントエンドのルーティング
Next.jsのルーティングはどのように設計しますか？

A) 単一ページ（/）でPhaseを状態管理
B) 各Phaseごとにルート（/phase1、/phase2、/phase3、/phase4）
C) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 9: アニメーション実装
暗転や抹消線のアニメーションはどのように実装しますか？

A) CSS Animations / Transitions
B) Framer Motion
C) React Spring
D) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 10: CDKスタックの構成
CDKのスタック構成はどのようにしますか？

A) 単一スタック（全リソースを1つのスタックに）
B) 複数スタック（Frontend用、Backend用、Shared用に分離）
C) その他（[Answer]: タグの後に説明してください）

[Answer]: A

---

すべての質問に回答したら、「完了しました」とお知らせください。
