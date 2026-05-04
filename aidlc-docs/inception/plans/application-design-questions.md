# アプリケーション設計 質問

要件定義・ユーザーストーリーを分析した結果、以下の設計判断について確認が必要です。
すべての [Answer]: タグに回答してください。

---

## Question 1: フロントエンドのコンポーネント分割方針
React SPA のコンポーネント構成をどのように分割しますか？

A) フェーズ単位（Phase1Page / Phase2Page / Phase3Page / Phase4Page の4コンポーネント）
B) 機能単位（Form / QuestionList / VerdictDisplay / ProfileResult など機能別に細分化）
C) A + B のハイブリッド（フェーズをページとして持ちつつ、内部を機能コンポーネントに分割）
X) Other（[Answer]: タグの後に説明してください）

[Answer]: 
A
---

## Question 2: フロントエンドの状態管理
4フェーズをまたぐデータ（Phase 1 の入力 → Phase 2 の質問 → Phase 3/4 の結果）をどう管理しますか？

A) React Context API（軽量、外部ライブラリ不要）
B) Redux Toolkit（大規模状態管理、ボイラープレート多め）
C) Zustand（軽量、シンプルな API）
D) useState + props drilling（最もシンプル、小規模向け）
X) Other（[Answer]: タグの後に説明してください）

[Answer]: 
B

---

## Question 3: バックエンドの Lambda 構成
Python Lambda をどのように構成しますか？

A) 単一 Lambda（`/generate-questions` と `/analyze` を1つの Lambda で処理）
B) 機能別 Lambda（`generate-questions` 用と `analyze` 用を別々の Lambda に分割）
X) Other（[Answer]: タグの後に説明してください）

[Answer]: 
B

---

## Question 4: Bedrock 呼び出しの設計
Bedrock（Claude Opus 4.7）への呼び出しをどのように設計しますか？

A) 同期呼び出し（Lambda が Bedrock のレスポンスを待ってから API Gateway に返す）
B) ストリーミング（Bedrock のストリーミングレスポンスをフロントエンドにリアルタイム転送）
X) Other（[Answer]: タグの後に説明してください）

[Answer]: 
A
---

## Question 5: データ永続化コンポーネント
匿名データの永続保存先を確定してください（要件では「DynamoDB or S3」と記載）。

A) DynamoDB（セッションIDをキーにした NoSQL、クエリが柔軟）
B) S3（JSON ファイルとして保存、シンプル・低コスト）
C) DynamoDB（メタデータ）+ S3（フルデータ）のハイブリッド
X) Other（[Answer]: タグの後に説明してください）

[Answer]: 
B
---

## Question 6: プロンプト管理の設計
Bedrock に渡すプロンプトテンプレートをどこで管理しますか？

A) Lambda コード内にハードコード（シンプル、変更時は再デプロイ）
B) S3 に外部化（プロンプトを S3 から読み込み、再デプロイなしで変更可能）
C) AWS Systems Manager Parameter Store（セキュアな設定管理）
X) Other（[Answer]: タグの後に説明してください）

[Answer]: 
B
---

## Question 7: CORS とセキュリティ
API Gateway の CORS 設定をどうしますか？

A) 開発中はすべてのオリジンを許可（`*`）、デモ本番では CloudFront ドメインのみ許可
B) 最初から CloudFront ドメインのみ許可（厳格）
X) Other（[Answer]: タグの後に説明してください）

[Answer]: 
A
---

## Question 8: 演出アニメーションの実装方針
暗転・抹消線・テキスト浮上などの演出をどのように実装しますか？

A) Pure CSS アニメーション（`@keyframes`、外部ライブラリ不要）
B) Framer Motion（React 向けアニメーションライブラリ、宣言的で書きやすい）
C) GSAP（高機能アニメーションライブラリ、複雑な演出に対応）
X) Other（[Answer]: タグの後に説明してください）

[Answer]: 
A
---
