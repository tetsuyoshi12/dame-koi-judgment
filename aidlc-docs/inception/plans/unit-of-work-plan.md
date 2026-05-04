# Unit of Work Plan - ダメ恋プロフィール裁判所

## ユニット分解アプローチ

このプランでは、以下の成果物を生成します：
- [x] unit-of-work.md - ユニット定義と責任範囲
- [x] unit-of-work-dependency.md - ユニット間の依存関係
- [x] unit-of-work-story-map.md - ストーリーとユニットのマッピング

## ユニット分解質問

以下の質問に回答してください。各質問の後の [Answer]: の後に、選択肢の記号（A、B、C等）を記入してください。

### Question 1: ユニット数の確認
Execution Planでは3ユニット（Frontend、Backend、Infrastructure）を推奨していますが、この構成で問題ありませんか？

A) 3ユニットで問題なし
B) 2ユニットに統合したい（Frontend+Backend、Infrastructure）
C) 4ユニット以上に分割したい
D) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 2: Frontend ユニットの責任範囲
Frontend ユニットの責任範囲を確認します。

A) 4フェーズのUI + 状態管理 + API通信 + 演出（すべて含む）
B) 4フェーズのUIのみ（API通信は別ユニット）
C) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 3: Backend ユニットの責任範囲
Backend ユニットの責任範囲を確認します。

A) Lambda関数 + Bedrockプロンプト実行 + バリデーション（すべて含む）
B) Lambda関数のみ（Bedrockプロンプトは別管理）
C) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 4: Infrastructure ユニットの責任範囲
Infrastructure ユニットの責任範囲を確認します。

A) CDKコード + デプロイスクリプト（すべて含む）
B) CDKコードのみ
C) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 5: ユニット間の開発順序
ユニットの開発順序をどのようにしますか？

A) 順次開発（Backend → Infrastructure → Frontend）
B) 並行開発（Backend と Frontend を並行、Infrastructure は最後）
C) 完全並行開発（3ユニットすべて同時）
D) その他（[Answer]: タグの後に説明してください）

[Answer]: B

### Question 6: API仕様の事前定義
Frontend と Backend を並行開発する場合、API仕様を事前に定義しますか？

A) はい、OpenAPI仕様書を作成
B) はい、TypeScript型定義を共有
C) はい、簡易的なドキュメントを作成
D) いいえ、Backend完成後にFrontendを開発
E) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 7: 共有コンポーネントの管理
型定義やユーティリティなど、複数ユニットで共有するコードはどのように管理しますか？

A) 各ユニットで独立して定義（重複OK）
B) 共有ディレクトリを作成（shared/ または common/）
C) npmパッケージとして分離
D) その他（[Answer]: タグの後に説明してください）

[Answer]: B

### Question 8: テストの責任範囲
各ユニットのテスト責任範囲を確認します。

A) 各ユニットが自身の単体テストを持つ
B) 統合テストは別ユニット（Test ユニット）として分離
C) テストはすべて手動（自動テストなし）
D) その他（[Answer]: タグの後に説明してください）

[Answer]: A

### Question 9: ドキュメントの管理
各ユニットのドキュメント（README等）はどのように管理しますか？

A) 各ユニットのディレクトリにREADME.mdを配置
B) aidlc-docs/ に統合ドキュメントとして管理
C) 両方（各ユニットのREADME + 統合ドキュメント）
D) その他（[Answer]: タグの後に説明してください）

[Answer]: B

### Question 10: デプロイの責任範囲
デプロイはどのユニットが担当しますか？

A) Infrastructure ユニットがすべてのデプロイを担当
B) 各ユニットが自身のデプロイスクリプトを持つ
C) CI/CDパイプラインで自動化（別途設定）
D) その他（[Answer]: タグの後に説明してください）

[Answer]: A

---

すべての質問に回答したら、「完了しました」とお知らせください。
