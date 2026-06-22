# エンジニアリング用語集 作成指示書

## 目的

開発・運用・設計・テスト・プロジェクト管理で現れる、
「現場でよく聞くが意味が曖昧になりやすい言葉」を整理した Markdown ベースの用語集を作成する。

単なる辞書ではなく、以下を重視する。

- 言葉の意味
- どのような場面で使われるか
- 似た言葉との違い
- 実際の現場でありがちな例
- 対策や回避策
- 関連用語へのリンク

---

## ディレクトリ構成

```text
.
├── README.md
├── docs/
│   └── authoring-guide.md
└── glossary/
    ├── design.md
    ├── concurrency.md
    ├── testing.md
    ├── operations.md
    ├── anti-patterns.md
    ├── bad-practices.md
    ├── project.md
    ├── aws-iac.md
    ├── database.md
    ├── distributed-systems.md
    ├── security.md
    ├── ai-ml.md
    └── templates/
        └── entry-template.md
```

---

## カテゴリ振り分けルール

用語が複数カテゴリに該当する場合、以下のルールで振り分ける。

1. **正（本文）は1箇所のみ** — 最も文脈が合うファイルに記述する
2. **他のファイルからは参照リンクで誘導** — 例: `※ [Snowflake Server](aws-iac.md#snowflake-server) を参照`
3. 判断基準: その用語が **最初に話題になる場面** を想像し、そのカテゴリに置く
4. 追加推奨カテゴリを将来作成する際も、既存ファイルの用語と重複する場合は参照リンクとする

---

## 各ファイルに含める主な用語

### design.md

- フールプルーフ
- フェイルセーフ
- フェイルソフト
- フェイルオープン
- フェイルクローズ
- ガードレール
- ディフェンス・イン・デプス
- サニティチェック
- イディオットプルーフ
- YAGNI
- Leaky Abstraction（抽象化の漏れ）
- Separation of Concerns
- Single Point of Control
- 冪等性（Idempotency）
- DRY
- KISS
- 結合度と凝集度（Coupling / Cohesion）
- 副作用（Side Effect）

### concurrency.md

- TOCTOU
- レースコンディション
- アトミック操作
- チェック・アンド・セット
- デッドロック
- ライブロック
- スターベーション
- ABA問題
- Thundering Herd（サンダリングハード）
- Priority Inversion（優先度逆転）
- Optimistic Locking / Pessimistic Locking

### testing.md

- フレーキーテスト
- スモークテスト
- サニティテスト
- リグレッションテスト
- ゴールデンテスト
- スナップショットテスト
- E2Eテスト
- ユニットテスト
- インテグレーションテスト
- Test Double（モック / スタブ / フェイク）
- カナリアテスト
- Heisenbug
- Mandelbug
- Property-based Testing
- Mutation Testing
- Test Pyramid
- Happy Path
- Sad Path（異常系）
- Edge Case（エッジケース）
- Corner Case（コーナーケース）
- Code Coverage（カバレッジ）
- Fixture
- Shift Left

### operations.md

- SPOF
- サーキットブレーカー
- バックプレッシャー
- バルクヘッド
- リトライストーム
- グレースフルデグラデーション
- レートリミット
- スロットリング
- ヘルスチェック
- リードネス
- ライブネス
- Canary Deploy
- Blue-Green Deployment
- Rolling Update
- Observability（オブザーバビリティ）
- Toil（トイル）
- Runbook
- Chaos Engineering
- Failover（フェイルオーバー）
- RTO / RPO
- Error Budget（エラーバジェット）
- On-call
- War Room
- ディザスタリカバリ（DR）
- DR戦略の4類型（Backup & Restore / Pilot Light / Warm Standby / Multi-Site）
- BCP（事業継続計画）
- Failback（フェイルバック）
- 3-2-1 バックアップルール
- Game Day（ゲームデー）

### anti-patterns.md

- Bike-shedding（自転車置き場の議論 / Parkinson's law of triviality）
- Yak Shaving
- Analysis Paralysis
- Premature Optimization
- Gold Plating
- NIH症候群
- Boiling the Ocean
- Scope Creep
- Feature Creep
- Golden Hammer
- Lava Flow（溶岩流コード）
- Big Ball of Mud
- Inner Platform Effect
- Accidental Complexity
- Resume Driven Development
- Abstraction Inversion
- Second System Effect
- Cargo Cult Agile
- Shiny Object Syndrome

### bad-practices.md

- バッドノウハウ
- 呪いのコピペ
- Cargo Cult Programming
- ショットガンデバッグ
- トライバルナレッジ
- 属人化
- ヒーロー運用
- Works on my machine
- YOLO Deploy
- ブロークンウィンドウ理論
- テクニカルデット
- マジックナンバー
- マジックストリング
- Shotgun Surgery
- Stringly Typed
- Primitive Obsession
- God Object / God Class
- Spaghetti Code
- Bus Factor（バス係数）
- Code Smell（コードスメル）
- Dead Code（デッドコード）
- Rubber Stamping（形骸化レビュー）
- nit / nitpick
- ※ [Snowflake Server](../glossary/aws-iac.md#snowflake-server) を参照
- ※ [Pet vs Cattle](../glossary/aws-iac.md#pet-vs-cattle) を参照

### project.md

- デスマーチ
- Parkinsonの法則
- 80/20ルール
- 5 Whys
- ラバーダックデバッグ
- ポストモーテム
- Blameless Postmortem
- ワークアラウンド
- ロールバック
- ミティゲーション
- 根本原因
- Brooks's Law
- Conway's Law
- Hofstadter's Law
- Sunk Cost Fallacy
- MTTR / MTTF / MTBF
- SLO / SLA / SLI
- Bystander Effect
- Velocity（ベロシティ）
- Retrospective（レトロスペクティブ）

### aws-iac.md

- ClickOps
- Drift（Configuration Drift / Terraform Drift）
- Immutable Infrastructure
- Blast Radius
- 手動変更
- Snowflake Server
- Pet vs Cattle
- Infrastructure as Code
- Day 2 Operations
- State File Locking
- Terraform Module Spaghetti
- Undocumented Resource
- Least Privilege (IAM)
- Terraform Plan/Apply の落とし穴
- タグ戦略（Tagging Strategy）

### database.md

- N+1 クエリ問題
- Slow Query
- Connection Pool Exhaustion
- Phantom Read / Dirty Read / Non-repeatable Read
- Hot Spot / Hot Key
- Index の効かないクエリ
- インデックスショットガン
- Migration の罠
- Replication Lag

### distributed-systems.md

- 結果整合性（Eventual Consistency）
- Split Brain
- CAP 定理
- Two Generals Problem
- Saga パターン
- Idempotency Key

### security.md

- Zero Trust
- SQL Injection
- XSS（Cross-Site Scripting）
- CSRF（Cross-Site Request Forgery）
- 認証と認可（Authentication vs Authorization）
- OAuth 2.0 / OIDC
- JWT（JSON Web Token）
- RBAC / ABAC
- Secret Rotation
- Supply Chain Attack
- CORS（Cross-Origin Resource Sharing）
- SSRF（Server-Side Request Forgery）
- 暗号化の基礎（Encryption at Rest / in Transit）
- ペネトレーションテスト
- Dependency Confusion
- Billion Laughs Attack
- XXE（XML External Entity）
- Zip Bomb
- ReDoS（Regular Expression DoS）
- カナリアトークン（Canarytoken / Honeytoken）
- ※ [Least Privilege (IAM)](../glossary/aws-iac.md#least-privilege-iam) を参照
- ※ [TOCTOU](../glossary/concurrency.md#toctou) を参照
- ※ [ディフェンス・イン・デプス](../glossary/design.md#ディフェンスインデプス) を参照
- ※ [フェイルオープン](../glossary/design.md#フェイルオープン) / [フェイルクローズ](../glossary/design.md#フェイルクローズ) を参照
- ※ [Shift Left](../glossary/testing.md#shift-left) を参照

### ai-ml.md

- Hallucination（ハルシネーション）
- Prompt Injection
- RAG（Retrieval-Augmented Generation）
- Embedding（エンベディング）
- Fine-tuning（ファインチューニング）
- Token（トークン）
- Temperature（テンプレチャー）
- Context Window（コンテキストウィンドウ）
- AI Guardrails
- Eval（評価）
- AI Agent（AIエージェント）
- Vibe Coding（バイブコーディング）
- MCP（Model Context Protocol）
- Overfitting（過学習）
- Bias（バイアス）
- Training vs Inference（学習と推論）

---

## 各用語のテンプレート

すべての用語は、以下のテンプレートに従って記述する。
同じテンプレートは `templates/entry-template.md` にも配置する。

```md
## 用語名

別名: 英語名 / 日本語別称

### 意味
1〜3行で簡潔に説明する。

### よくある例
- 実際の現場での例を2〜3個
- AWS / Terraform / systemd / CI / Rails / Docker / Kubernetes など、現実的な文脈を優先

### ありがちな症状
- どういう時にこの言葉を思い出すべきか
- 「こうなっていたら怪しい」という兆候

### 近い言葉との違い
- 関連用語との差を箇条書きで示す
- 2〜5個程度

### 背景・語源（該当する用語のみ）
- 用語の由来、命名のエピソード、初出の文献など
- 語源に面白いストーリーがある場合や、名前の由来を知ることで理解が深まる場合に記載する
- すべての用語に必須ではない

### 対策
- 避け方
- 発生したときの対処
- 実務でのベストプラクティス

### 関連用語
- [別の用語](file.md#anchor)
- [別の用語](file.md#anchor)
```

---

## 記述スタイル

- 簡潔に、しかし実務で使えるレベルまで書く
- 1項目あたり 10〜30 行程度を目安にする
- 定義だけで終わらず、「どういう時に使うか」を必ず書く
- 難しい理論よりも、現場での具体例を優先する
- AWS / Terraform / ECS / systemd / CI / Rails / Docker など、実際に遭遇しやすい文脈を多く入れる
- 可能なら「悪い例」と「良い例」を並べる
- 関連用語リンクは Markdown 記法 `[用語](file.md#anchor)` で統一する

例:

```md
悪い例:
- CI が不安定なので `sleep 30` を追加

良い例:
- readiness check や待機条件を導入
```

---

## README.md の構成

README.md は索引として使う。全カテゴリについて記載すること。

```md
# エンジニアリング現場の用語集

## 設計
- [フールプルーフ](glossary/design.md#フールプルーフ)
- [フェイルセーフ](glossary/design.md#フェイルセーフ)
- [YAGNI](glossary/design.md#yagni)

## 並行処理
- [TOCTOU](glossary/concurrency.md#toctou)
...

## データベース
- [N+1 クエリ問題](glossary/database.md#n1-クエリ問題)
...

## 分散システム
- [結果整合性](glossary/distributed-systems.md#結果整合性)
...

## よく一緒に現れる用語
- ClickOps → Snowflake Server → Drift → Undocumented Resource
- TOCTOU → レースコンディション → アトミック操作 → Optimistic Locking
...
```

---

## 初期作成時の優先順位

### Phase 1（現場で即効性が高い）

1. anti-patterns.md
2. bad-practices.md
3. testing.md
4. concurrency.md
5. aws-iac.md

理由:
- 日常会話で最も遭遇しやすい
- 意味を知らないと会議やレビューで困る
- AWS / Terraform / CI / systemd の現場で即役に立つ

### Phase 2（基盤となる知識）

6. operations.md
7. design.md
8. project.md

理由:
- operations.md は運用現場のデプロイ戦略・障害対応で必須
- design.md は設計レビューの共通語彙
- project.md の法則群はマネジメント層との議論に有用

---

## 追加推奨カテゴリ

将来的に必要になったら、以下のファイルを追加する。
既存ファイルの用語と重複する場合は、正は既存ファイルに残し、新規ファイルからは参照リンクとする。

```text
network.md
systemd.md
kubernetes.md
```

例:

- systemd.md
  - Watchdog
  - READY=1
  - TimeoutStartSec
  - KillMode
  - Restart Loop

- kubernetes.md
  - CrashLoopBackOff
  - Liveness Probe — ※ [operations.md](../glossary/operations.md#ライブネス) を参照
  - Readiness Probe — ※ [operations.md](../glossary/operations.md#リードネス) を参照
  - Pod Eviction
