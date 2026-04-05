# エンジニアリング現場の用語集

開発・運用・設計・テスト・プロジェクト管理で現れる「現場でよく聞くが意味が曖昧になりやすい言葉」を整理した用語集です。

単なる定義集ではなく、**どういう場面で使うか**、**似た言葉との違い**、**現場でありがちな例**、**対策**まで踏み込んでいます。

## 構成

```
glossary/
├── design.md              設計原則・安全設計
├── concurrency.md         並行処理・排他制御
├── testing.md             テスト手法・テスト戦略
├── operations.md          運用・デプロイ・障害対応
├── anti-patterns.md       アンチパターン
├── bad-practices.md       悪い習慣・コードの匂い
├── project.md             プロジェクト管理・法則
├── aws-iac.md             AWS / Infrastructure as Code
├── database.md            データベース
├── distributed-systems.md 分散システム
└── templates/
    └── entry-template.md  用語エントリのテンプレート
```

各用語は共通のテンプレートに従い、**意味 → よくある例 → ありがちな症状 → 近い言葉との違い → 背景・語源 → 対策 → 関連用語** の順で記述しています。

執筆ガイドラインは [docs/authoring-guide.md](docs/authoring-guide.md) を参照してください。

---

## 用語一覧

### 設計

- [フールプルーフ](glossary/design.md#フールプルーフ)
- [フェイルセーフ](glossary/design.md#フェイルセーフ)
- [フェイルソフト](glossary/design.md#フェイルソフト)
- [フェイルオープン](glossary/design.md#フェイルオープン)
- [フェイルクローズ](glossary/design.md#フェイルクローズ)
- [ガードレール](glossary/design.md#ガードレール)
- [ディフェンス・イン・デプス](glossary/design.md#ディフェンスインデプス)
- [サニティチェック](glossary/design.md#サニティチェック)
- [イディオットプルーフ](glossary/design.md#イディオットプルーフ)
- [YAGNI](glossary/design.md#yagni)
- [Leaky Abstraction](glossary/design.md#leaky-abstraction抽象化の漏れ)
- [Separation of Concerns](glossary/design.md#separation-of-concerns)
- [Single Point of Control](glossary/design.md#single-point-of-control)
- [冪等性（Idempotency）](glossary/design.md#冪等性idempotency)
- [DRY](glossary/design.md#dry)
- [KISS](glossary/design.md#kiss)
- [結合度と凝集度（Coupling / Cohesion）](glossary/design.md#結合度と凝集度coupling--cohesion)
- [副作用（Side Effect）](glossary/design.md#副作用side-effect)

### 並行処理

- [TOCTOU](glossary/concurrency.md#toctou)
- [レースコンディション](glossary/concurrency.md#レースコンディション)
- [アトミック操作](glossary/concurrency.md#アトミック操作)
- [チェック・アンド・セット](glossary/concurrency.md#チェックアンドセット)
- [デッドロック](glossary/concurrency.md#デッドロック)
- [ライブロック](glossary/concurrency.md#ライブロック)
- [スターベーション](glossary/concurrency.md#スターベーション)
- [ABA問題](glossary/concurrency.md#aba問題)
- [Thundering Herd](glossary/concurrency.md#thundering-herdサンダリングハード)
- [Priority Inversion](glossary/concurrency.md#priority-inversion優先度逆転)
- [Optimistic / Pessimistic Locking](glossary/concurrency.md#optimistic-locking--pessimistic-locking)

### テスト

- [フレーキーテスト](glossary/testing.md#フレーキーテスト)
- [スモークテスト](glossary/testing.md#スモークテスト)
- [サニティテスト](glossary/testing.md#サニティテスト)
- [リグレッションテスト](glossary/testing.md#リグレッションテスト)
- [ゴールデンテスト](glossary/testing.md#ゴールデンテスト)
- [スナップショットテスト](glossary/testing.md#スナップショットテスト)
- [E2Eテスト](glossary/testing.md#e2eテスト)
- [ユニットテスト](glossary/testing.md#ユニットテスト)
- [インテグレーションテスト](glossary/testing.md#インテグレーションテスト)
- [Test Double（モック / スタブ / フェイク）](glossary/testing.md#test-doubleモック--スタブ--フェイク)
- [カナリアテスト](glossary/testing.md#カナリアテスト)
- [Heisenbug](glossary/testing.md#heisenbug)
- [Mandelbug](glossary/testing.md#mandelbug)
- [Property-based Testing](glossary/testing.md#property-based-testing)
- [Mutation Testing](glossary/testing.md#mutation-testing)
- [Test Pyramid](glossary/testing.md#test-pyramid)
- [Happy Path](glossary/testing.md#happy-path)
- [Sad Path](glossary/testing.md#sad-path)
- [Edge Case](glossary/testing.md#edge-case)
- [Corner Case](glossary/testing.md#corner-case)
- [Code Coverage（カバレッジ）](glossary/testing.md#code-coverageカバレッジ)
- [Fixture](glossary/testing.md#fixture)
- [Shift Left](glossary/testing.md#shift-left)

### 運用

- [SPOF](glossary/operations.md#spof)
- [サーキットブレーカー](glossary/operations.md#サーキットブレーカー)
- [バックプレッシャー](glossary/operations.md#バックプレッシャー)
- [バルクヘッド](glossary/operations.md#バルクヘッド)
- [リトライストーム](glossary/operations.md#リトライストーム)
- [グレースフルデグラデーション](glossary/operations.md#グレースフルデグラデーション)
- [レートリミット](glossary/operations.md#レートリミット)
- [スロットリング](glossary/operations.md#スロットリング)
- [ヘルスチェック](glossary/operations.md#ヘルスチェック)
- [リードネス](glossary/operations.md#リードネス)
- [ライブネス](glossary/operations.md#ライブネス)
- [Canary Deploy](glossary/operations.md#canary-deploy)
- [Blue-Green Deployment](glossary/operations.md#blue-green-deployment)
- [Rolling Update](glossary/operations.md#rolling-update)
- [Observability](glossary/operations.md#observabilityオブザーバビリティ)
- [Toil](glossary/operations.md#toilトイル)
- [Runbook](glossary/operations.md#runbook)
- [Chaos Engineering](glossary/operations.md#chaos-engineering)
- [Failover](glossary/operations.md#failoverフェイルオーバー)
- [RTO / RPO](glossary/operations.md#rto--rpo)
- [Error Budget](glossary/operations.md#error-budgetエラーバジェット)
- [On-call](glossary/operations.md#on-call)
- [War Room](glossary/operations.md#war-room)

### アンチパターン

- [Bike-shedding](glossary/anti-patterns.md#bike-shedding)
- [Yak Shaving](glossary/anti-patterns.md#yak-shaving)
- [Analysis Paralysis](glossary/anti-patterns.md#analysis-paralysis)
- [Premature Optimization](glossary/anti-patterns.md#premature-optimization)
- [Gold Plating](glossary/anti-patterns.md#gold-plating)
- [NIH症候群](glossary/anti-patterns.md#nih症候群)
- [Boiling the Ocean](glossary/anti-patterns.md#boiling-the-ocean)
- [Scope Creep](glossary/anti-patterns.md#scope-creep)
- [Feature Creep](glossary/anti-patterns.md#feature-creep)
- [Golden Hammer](glossary/anti-patterns.md#golden-hammer)
- [Lava Flow](glossary/anti-patterns.md#lava-flow)
- [Big Ball of Mud](glossary/anti-patterns.md#big-ball-of-mud)
- [Inner Platform Effect](glossary/anti-patterns.md#inner-platform-effect)
- [Accidental Complexity](glossary/anti-patterns.md#accidental-complexity)
- [Resume Driven Development](glossary/anti-patterns.md#resume-driven-development)
- [Abstraction Inversion](glossary/anti-patterns.md#abstraction-inversion)
- [Second System Effect](glossary/anti-patterns.md#second-system-effect)
- [Cargo Cult Agile](glossary/anti-patterns.md#cargo-cult-agile)
- [Shiny Object Syndrome](glossary/anti-patterns.md#shiny-object-syndrome)

### 悪い習慣

- [バッドノウハウ](glossary/bad-practices.md#バッドノウハウ)
- [呪いのコピペ](glossary/bad-practices.md#呪いのコピペ)
- [Cargo Cult Programming](glossary/bad-practices.md#cargo-cult-programming)
- [ショットガンデバッグ](glossary/bad-practices.md#ショットガンデバッグ)
- [トライバルナレッジ](glossary/bad-practices.md#トライバルナレッジ)
- [属人化](glossary/bad-practices.md#属人化)
- [ヒーロー運用](glossary/bad-practices.md#ヒーロー運用)
- [Works on my machine](glossary/bad-practices.md#works-on-my-machine)
- [YOLO Deploy](glossary/bad-practices.md#yolo-deploy)
- [ブロークンウィンドウ理論](glossary/bad-practices.md#ブロークンウィンドウ理論)
- [テクニカルデット](glossary/bad-practices.md#テクニカルデット)
- [マジックナンバー](glossary/bad-practices.md#マジックナンバー)
- [マジックストリング](glossary/bad-practices.md#マジックストリング)
- [Shotgun Surgery](glossary/bad-practices.md#shotgun-surgery)
- [Stringly Typed](glossary/bad-practices.md#stringly-typed)
- [Primitive Obsession](glossary/bad-practices.md#primitive-obsession)
- [God Object / God Class](glossary/bad-practices.md#god-object--god-class)
- [Spaghetti Code](glossary/bad-practices.md#spaghetti-code)
- [Bus Factor](glossary/bad-practices.md#bus-factorバス係数)
- [Code Smell](glossary/bad-practices.md#code-smellコードスメル)
- [Dead Code](glossary/bad-practices.md#dead-codeデッドコード)
- [Rubber Stamping](glossary/bad-practices.md#rubber-stamping形骸化レビュー)

### プロジェクト管理

- [デスマーチ](glossary/project.md#デスマーチ)
- [Parkinsonの法則](glossary/project.md#parkinsonの法則)
- [80/20ルール](glossary/project.md#8020ルール)
- [5 Whys](glossary/project.md#5-whys)
- [ラバーダックデバッグ](glossary/project.md#ラバーダックデバッグ)
- [ポストモーテム](glossary/project.md#ポストモーテム)
- [Blameless Postmortem](glossary/project.md#blameless-postmortem)
- [ワークアラウンド](glossary/project.md#ワークアラウンド)
- [ロールバック](glossary/project.md#ロールバック)
- [ミティゲーション](glossary/project.md#ミティゲーション)
- [根本原因](glossary/project.md#根本原因)
- [Brooks's Law](glossary/project.md#brookss-law)
- [Conway's Law](glossary/project.md#conways-law)
- [Hofstadter's Law](glossary/project.md#hofstadters-law)
- [Sunk Cost Fallacy](glossary/project.md#sunk-cost-fallacy)
- [MTTR / MTTF / MTBF](glossary/project.md#mttr--mttf--mtbf)
- [SLO / SLA / SLI](glossary/project.md#slo--sla--sli)
- [Bystander Effect](glossary/project.md#bystander-effect)
- [Velocity](glossary/project.md#velocityベロシティ)
- [Retrospective](glossary/project.md#retrospectiveレトロスペクティブ)

### AWS / IaC

- [ClickOps](glossary/aws-iac.md#clickops)
- [Drift](glossary/aws-iac.md#drift)
- [Immutable Infrastructure](glossary/aws-iac.md#immutable-infrastructure)
- [Blast Radius](glossary/aws-iac.md#blast-radius)
- [手動変更](glossary/aws-iac.md#手動変更)
- [Snowflake Server](glossary/aws-iac.md#snowflake-server)
- [Pet vs Cattle](glossary/aws-iac.md#pet-vs-cattle)
- [Infrastructure as Code](glossary/aws-iac.md#infrastructure-as-code)
- [Day 2 Operations](glossary/aws-iac.md#day-2-operations)
- [State File Locking](glossary/aws-iac.md#state-file-locking)
- [Terraform Module Spaghetti](glossary/aws-iac.md#terraform-module-spaghetti)
- [Undocumented Resource](glossary/aws-iac.md#undocumented-resource)
- [Least Privilege (IAM)](glossary/aws-iac.md#least-privilege-iam)
- [Terraform Plan/Apply の落とし穴](glossary/aws-iac.md#terraform-planapply-の落とし穴)
- [タグ戦略（Tagging Strategy）](glossary/aws-iac.md#タグ戦略tagging-strategy)

### データベース

- [N+1 クエリ問題](glossary/database.md#n1-クエリ問題)
- [Slow Query](glossary/database.md#slow-query)
- [Connection Pool Exhaustion](glossary/database.md#connection-pool-exhaustion)
- [Phantom Read / Dirty Read / Non-repeatable Read](glossary/database.md#phantom-read--dirty-read--non-repeatable-read)
- [Hot Spot / Hot Key](glossary/database.md#hot-spot--hot-key)
- [Index の効かないクエリ](glossary/database.md#index-の効かないクエリ)
- [Migration の罠](glossary/database.md#migration-の罠)
- [Replication Lag](glossary/database.md#replication-lag)

### 分散システム

- [結果整合性](glossary/distributed-systems.md#結果整合性)
- [Split Brain](glossary/distributed-systems.md#split-brain)
- [CAP 定理](glossary/distributed-systems.md#cap-定理)
- [Two Generals Problem](glossary/distributed-systems.md#two-generals-problem)
- [Saga パターン](glossary/distributed-systems.md#saga-パターン)
- [Idempotency Key](glossary/distributed-systems.md#idempotency-key)

---

## よく一緒に現れる用語

- ClickOps → Snowflake Server → Drift → Undocumented Resource
- TOCTOU → レースコンディション → アトミック操作 → Optimistic Locking
- フレーキーテスト → Heisenbug → Works on my machine
- Bike-shedding → Analysis Paralysis → Scope Creep → Sunk Cost Fallacy
- ヒーロー運用 → トライバルナレッジ → バッドノウハウ → Bus Factor
- リトライストーム → Thundering Herd → サーキットブレーカー → バックプレッシャー
- Lava Flow → Big Ball of Mud → テクニカルデット → ブロークンウィンドウ理論
- デスマーチ → Brooks's Law → Parkinsonの法則 → Hofstadter's Law
- Canary Deploy → Blue-Green Deployment → Rolling Update → Blast Radius
- God Object → Spaghetti Code → Shotgun Surgery → Accidental Complexity
- 属人化 → Runbook → Toil → Bystander Effect
- N+1 クエリ問題 → Slow Query → Index の効かないクエリ → Connection Pool Exhaustion
- 結果整合性 → Replication Lag → Split Brain → CAP 定理
- 冪等性 → Idempotency Key → Saga パターン → 結果整合性
- Code Smell → Dead Code → テクニカルデット → Rubber Stamping
- RTO / RPO → Failover → Error Budget → SLO / SLA / SLI
- Cargo Cult Agile → Shiny Object Syndrome → Second System Effect

---

## ライセンス

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

この用語集は [CC0 1.0 Universal (Public Domain Dedication)](https://creativecommons.org/publicdomain/zero/1.0/) の下で公開されています。
