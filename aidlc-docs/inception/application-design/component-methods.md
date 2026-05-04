# コンポーネントメソッド定義

*注: 詳細なビジネスロジック・バリデーションルールは CONSTRUCTION フェーズの Functional Design で定義する*

---

## フロントエンドメソッド

### FE-01: App
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `renderCurrentPhase()` | `currentPhase: number (1-4)` | `JSX.Element` | 現在のフェーズに対応するページコンポーネントを返す |

### FE-02: Phase1Page
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `handleInputChange(field, value)` | `field: string, value: string` | `void` | フォーム入力値をローカル state に反映する |
| `validateForm()` | `formData: Phase1FormData` | `ValidationResult` | 空欄・文字数不足をチェックし、エラーメッセージを返す |
| `handleSubmit()` | `formData: Phase1FormData` | `void` | バリデーション後に Redux Store へ保存し Phase 2 へ遷移する |

### FE-03: Phase2Page
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `fetchQuestions(phase1Data)` | `phase1Data: Phase1FormData` | `Promise<Question[]>` | `/generate-questions` API を呼び出し本音質問リストを取得する |
| `renderQuestionInput(question)` | `question: Question` | `JSX.Element` | 質問タイプ（slider/text）に応じた入力UIを返す |
| `handleAnswerChange(questionId, value)` | `questionId: string, value: string \| number` | `void` | 回答値をローカル state に反映する |
| `validateAllAnswered()` | `answers: AnswerMap` | `boolean` | 全質問に回答済みかチェックする |
| `handleSubmit()` | `answers: AnswerMap` | `void` | Redux Store へ保存し Phase 3 へ遷移する |

### FE-04: Phase3Page
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `triggerAnalysis()` | `sessionData: SessionData` | `Promise<AnalysisResult>` | `/analyze` API を呼び出し分析結果を取得する |
| `playVerdictAnimation()` | `verdictText: string` | `void` | 暗転 → 無音 → テキスト浮上のアニメーションシーケンスを開始する |
| `onAnimationComplete()` | `void` | `void` | アニメーション完了後に「Phase 4 へ進む」ボタンを有効化する |

### FE-05: Phase4Page
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `playStrikethroughAnimation()` | `originalTexts: string[], newTexts: string[]` | `void` | 虚飾テキストへの抹消線 → 改善案上書きアニメーションを再生する |
| `handleCopyProfile()` | `profileText: string` | `void` | 新プロフィール文をクリップボードにコピーし、コピー完了フィードバックを表示する |
| `skipAnimation()` | `void` | `void` | アニメーションをスキップして最終結果を即時表示する |

### FE-06: MonolithCharacter
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `renderState(state)` | `state: 'idle' \| 'analyzing' \| 'verdict' \| 'issuing'` | `JSX.Element` | フェーズに応じたモノリスの状態ビジュアルを返す |

---

## バックエンドメソッド

### BE-01: GenerateQuestionsLambda
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `handler(event, context)` | `APIGatewayEvent, LambdaContext` | `APIGatewayResponse` | Lambda エントリーポイント。リクエストを受け取り質問リストを返す |
| `load_prompt_template(s3_client, bucket, key)` | `s3_client, bucket: str, key: str` | `str` | S3 からプロンプトテンプレートを読み込む |
| `build_question_prompt(template, phase1_data)` | `template: str, phase1_data: dict` | `str` | Phase 1 データをテンプレートに埋め込みプロンプトを構築する |
| `call_bedrock(bedrock_client, prompt)` | `bedrock_client, prompt: str` | `dict` | Bedrock（Claude Opus 4.7）を呼び出し質問リストを取得する |
| `parse_questions_response(raw_response)` | `raw_response: str` | `list[Question]` | Bedrock のレスポンスをパースし Question オブジェクトのリストに変換する |

### BE-02: AnalyzeLambda
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `handler(event, context)` | `APIGatewayEvent, LambdaContext` | `APIGatewayResponse` | Lambda エントリーポイント。Phase 1/2 データを受け取り分析結果を返す |
| `load_prompt_template(s3_client, bucket, key)` | `s3_client, bucket: str, key: str` | `str` | S3 からプロンプトテンプレートを読み込む |
| `build_analysis_prompt(template, phase1_data, phase2_data)` | `template: str, phase1_data: dict, phase2_data: dict` | `str` | Phase 1/2 の対応ペアをテンプレートに埋め込みプロンプトを構築する |
| `call_bedrock(bedrock_client, prompt)` | `bedrock_client, prompt: str` | `dict` | Bedrock（Claude Opus 4.7）を呼び出し分析結果を取得する |
| `parse_analysis_response(raw_response)` | `raw_response: str` | `AnalysisResult` | Bedrock のレスポンスをパースし AnalysisResult オブジェクトに変換する |
| `save_session_data(s3_client, bucket, session_id, data)` | `s3_client, bucket: str, session_id: str, data: dict` | `None` | 匿名化セッションデータを S3 に保存する |
| `generate_session_id()` | `void` | `str` | UUID v4 でセッションIDを生成する（個人識別情報を含まない） |
