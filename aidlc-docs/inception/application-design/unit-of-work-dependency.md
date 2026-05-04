# Unit of Work 依存関係 - ダメ恋プロフィール裁判所

## 依存関係マトリクス

| From \ To | Frontend | Backend | Infrastructure | Shared |
|-----------|----------|---------|----------------|--------|
| **Frontend** | - | API経由 | - | 型定義参照 |
| **Backend** | - | - | - | 型定義参照 |
| **Infrastructure** | デプロイ先設定 | Lambda関数コード | - | - |
| **Shared** | - | - | - | - |

### 依存関係の詳細

#### Frontend → Backend
- **依存タイプ**: ランタイム依存（API経由）
- **依存内容**: REST API呼び出し（POST /api/analyze）
- **結合度**: 疎結合（OpenAPI仕様で定義）

#### Frontend → Shared
- **依存タイプ**: コンパイル時依存
- **依存内容**: 型定義（Phase1Data, Phase2Data, AnalysisResult等）
- **結合度**: 密結合（型定義の変更は影響大）

#### Backend → Shared
- **依存タイプ**: コンパイル時依存
- **依存内容**: 型定義（Phase1Data, Phase2Data, AnalysisResult等）
- **結合度**: 密結合（型定義の変更は影響大）

#### Infrastructure → Backend
- **依存タイプ**: ビルド時依存
- **依存内容**: Lambda関数のビルド成果物
- **結合度**: 密結合（Backend のビルドが必要）

#### Infrastructure → Frontend
- **依存タイプ**: 設定依存
- **依存内容**: API Gateway URL（環境変数として設定）
- **結合度**: 疎結合（URL文字列のみ）

---

## データフロー図

```
[ユーザー]
    |
    | 入力
    v
[Frontend Unit]
    |
    | HTTP POST /api/analyze
    | (Phase1Data + Phase2Data)
    v
[API Gateway] ← [Infrastructure Unit がデプロイ]
    |
    | Lambda トリガー
    v
[Backend Unit: Lambda関数]
    |
    +--→ [Parameter Store] (プロンプト取得)
    |
    +--→ [Amazon Bedrock] (AI分析)
    |
    +--→ [CloudWatch Logs] (ログ記録)
    |
    | HTTP Response
    | (AnalysisResult)
    v
[Frontend Unit]
    |
    | 表示
    v
[ユーザー]
```

---

## 開発時の依存関係

### Phase 1: 準備（Day 1）

```
[Shared Unit]
    |
    +--→ 型定義作成
    |
    +--→ OpenAPI仕様書作成
    |
    v
[Frontend Unit] と [Backend Unit] が参照可能
```

**依存関係**:
- Frontend と Backend は Shared の型定義に依存
- 両ユニットは OpenAPI 仕様書に従って開発

### Phase 2: 並行開発（Day 2-11）

```
[Frontend Unit]                    [Backend Unit]
    |                                  |
    | (OpenAPI仕様に基づく)             | (OpenAPI仕様に基づく)
    |                                  |
    +--→ [Shared/types] ←--------------+
    |                                  |
    +--→ [Shared/openapi] ←------------+
```

**依存関係**:
- Frontend と Backend は独立して開発可能
- 両ユニットは Shared の型定義を参照
- OpenAPI 仕様書が契約として機能

### Phase 3: 統合（Day 12-13）

```
[Backend Unit]
    |
    | ビルド成果物
    v
[Infrastructure Unit]
    |
    | CDK デプロイ
    v
[AWS環境]
    |
    | API Gateway URL
    v
[Frontend Unit]
    |
    | 環境変数設定
    v
[統合テスト]
```

**依存関係**:
- Infrastructure は Backend のビルド成果物に依存
- Frontend は Infrastructure がデプロイした API Gateway URL に依存

---

## ビルド依存関係

### ビルド順序

1. **Shared**: 型定義とOpenAPI仕様書（最初に確定）
2. **Backend**: Lambda関数のビルド（TypeScript → JavaScript）
3. **Infrastructure**: CDKスタックのデプロイ（Backend のビルド成果物を使用）
4. **Frontend**: Next.jsアプリのビルド（API URLを環境変数に設定）

### ビルドコマンド

**Shared**:
```bash
# 型定義の検証
cd shared
npx tsc --noEmit
```

**Backend**:
```bash
cd backend
npm run build        # TypeScript → JavaScript
npm run test         # 単体テスト実行
```

**Infrastructure**:
```bash
cd infrastructure
npm run build        # CDK スタックのビルド
cdk synth            # CloudFormation テンプレート生成
cdk deploy           # AWS へデプロイ
```

**Frontend**:
```bash
cd frontend
export NEXT_PUBLIC_API_URL=<API Gateway URL>
npm run build        # Next.js ビルド
npm run test         # 単体テスト実行
```

---

## ランタイム依存関係

### Frontend ランタイム依存

```
[Frontend (ブラウザ)]
    |
    | fetch API
    v
[API Gateway]
    |
    v
[Backend (Lambda)]
```

**依存サービス**:
- API Gateway（Infrastructure Unit がデプロイ）
- Backend Lambda関数

### Backend ランタイム依存

```
[Backend (Lambda)]
    |
    +--→ [Parameter Store] (プロンプト取得)
    |
    +--→ [Amazon Bedrock] (AI分析)
    |
    +--→ [CloudWatch Logs] (ログ記録)
```

**依存サービス**:
- AWS Systems Manager Parameter Store
- Amazon Bedrock
- Amazon CloudWatch Logs

---

## 環境変数依存

### Frontend 環境変数

| 変数名 | 依存元 | 設定タイミング |
|--------|--------|--------------|
| `NEXT_PUBLIC_API_URL` | Infrastructure Unit（API Gateway URL） | Infrastructure デプロイ後 |

### Backend 環境変数

| 変数名 | 依存元 | 設定タイミング |
|--------|--------|--------------|
| `AWS_REGION` | Infrastructure Unit（CDK設定） | Infrastructure デプロイ時 |
| `BEDROCK_MODEL_ID` | Infrastructure Unit（CDK設定） | Infrastructure デプロイ時 |
| `PROMPT_PARAMETER_NAME` | Infrastructure Unit（CDK設定） | Infrastructure デプロイ時 |

---

## 循環依存の回避

### 回避策1: OpenAPI仕様による契約

```
[Frontend] ←--OpenAPI仕様--→ [Backend]
```

- Frontend と Backend は直接依存せず、OpenAPI仕様を契約として使用
- 両ユニットは独立して開発可能

### 回避策2: Shared ユニットの分離

```
[Shared]
    |
    +--→ [Frontend]
    |
    +--→ [Backend]
```

- 共通の型定義を Shared ユニットに集約
- Frontend と Backend は Shared に依存するが、相互依存は発生しない

### 回避策3: Infrastructure の後方配置

```
[Backend] --ビルド成果物--> [Infrastructure]
[Frontend] <--API URL-- [Infrastructure]
```

- Infrastructure は Backend のビルド成果物を使用
- Frontend は Infrastructure がデプロイした API URL を使用
- 循環依存は発生しない

---

## 依存関係の変更管理

### Shared 型定義の変更

**影響範囲**: Frontend, Backend

**変更手順**:
1. Shared の型定義を変更
2. Frontend と Backend のコンパイルエラーを確認
3. 影響を受けるコードを修正
4. 単体テストを実行
5. 統合テストを実行

### OpenAPI 仕様の変更

**影響範囲**: Frontend, Backend

**変更手順**:
1. OpenAPI 仕様書を変更
2. Frontend の API呼び出しコードを確認
3. Backend のハンドラーコードを確認
4. 両ユニットのコードを修正
5. 統合テストを実行

### Backend ビルド成果物の変更

**影響範囲**: Infrastructure

**変更手順**:
1. Backend のコードを変更
2. Backend をビルド
3. Infrastructure を再デプロイ
4. 統合テストを実行

---

## テスト時の依存関係

### 単体テスト

**Frontend 単体テスト**:
- 依存: なし（モック使用）
- テスト対象: ValidationService, ClipboardService, ApiService

**Backend 単体テスト**:
- 依存: なし（モック使用）
- テスト対象: BedrockService, PromptService, ValidationService, ErrorService

### 統合テスト

**Frontend + Backend 統合テスト**:
- 依存: Infrastructure Unit（デプロイ済み環境）
- テスト対象: E2Eフロー（Phase 1 → Phase 2 → Phase 3 → Phase 4）

---

## デプロイ時の依存関係

### デプロイ順序

1. **Parameter Store**: プロンプトテンプレートを保存（Infrastructure Unit）
2. **Backend Lambda**: Lambda関数をデプロイ（Infrastructure Unit）
3. **API Gateway**: REST APIをデプロイ（Infrastructure Unit）
4. **Frontend**: API URLを設定してデプロイ（Frontend Unit）

### ロールバック順序

1. **Frontend**: 前バージョンに戻す
2. **Infrastructure**: CloudFormation スタックをロールバック
3. **Backend**: 前バージョンの Lambda関数に戻す

---

## 依存関係の可視化

### 開発時依存関係図

```
        [Shared]
           |
           +------------------+
           |                  |
           v                  v
      [Frontend]          [Backend]
           |                  |
           |                  v
           |          [Infrastructure]
           |                  |
           +------------------+
                     |
                     v
              [統合テスト]
```

### ランタイム依存関係図

```
[ユーザー]
    |
    v
[Frontend]
    |
    v
[API Gateway]
    |
    v
[Backend Lambda]
    |
    +--→ [Parameter Store]
    |
    +--→ [Bedrock]
    |
    +--→ [CloudWatch Logs]
```

---

## 依存関係管理のベストプラクティス

### 1. 型定義の一元管理
- Shared ユニットで型定義を一元管理
- Frontend と Backend で型定義を重複させない

### 2. OpenAPI仕様の活用
- API仕様を事前に定義
- Frontend と Backend の並行開発を可能にする

### 3. 環境変数の明示的な管理
- 各ユニットの環境変数を明確に文書化
- Infrastructure Unit で環境変数を一元管理

### 4. ビルド順序の遵守
- Shared → Backend → Infrastructure → Frontend の順序を守る
- 依存関係を無視したビルドは失敗の原因

### 5. バージョン管理
- 各ユニットのバージョンを明示的に管理
- 互換性のないバージョン間の依存を避ける
