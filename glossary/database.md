# データベース

---

## N+1 クエリ問題

別名: N+1 Query Problem / N+1 問題

### 意味
ORM が関連レコードを1件ずつ取得するために、一覧取得の1回 + 各レコードの関連取得 N 回で合計 N+1 回のクエリが発行される性能問題。データ件数に比例してクエリ数が線形に増加し、レスポンスタイムが劣化する。

### よくある例
- Rails で `Post.all.each { |p| p.comments }` と書くと、投稿一覧の取得に1回 + 各投稿のコメント取得に N 回のクエリが走る
- Django の `Article.objects.all()` をテンプレートでループし、各記事の `article.author.name` にアクセスするたびに著者テーブルへ SELECT が飛ぶ
- GraphQL のリゾルバで関連エンティティを個別に解決し、DataLoader を導入していないために大量のクエリが発生する
- ActiveAdmin の一覧画面で関連カラムを表示し、開発環境では気づかないが本番の数千件で致命的に遅くなる

### ありがちな症状
- 開発環境（少量データ）では快適だが、本番（大量データ）でレスポンスが数秒〜数十秒に膨れ上がる
- `rails console` や Django Debug Toolbar でクエリ数を見ると数百〜数千件出ている
- PostgreSQL / MySQL の `max_connections` に迫る接続数になる
- New Relic や Datadog の APM で DB 待ち時間が支配的になっている

### 近い言葉との違い
- [Slow Query](#slow-query): N+1 は個々のクエリは高速でも、総数が問題になる。Slow Query は1本のクエリ自体が遅い
- [Connection Pool Exhaustion](#connection-pool-exhaustion): N+1 の大量クエリが接続プールを圧迫し、枯渇を引き起こすことがある
- [Index の効かないクエリ](#index-の効かないクエリ): N+1 の個々のクエリにインデックスが効いていなければ、問題はさらに深刻化する

### 背景・語源
ORM（Object-Relational Mapping）の利便性と引き換えに発生する典型的な性能問題。Ruby on Rails の ActiveRecord が普及した2000年代後半から広く認知されるようになった。「N+1」という名前は、1回のリストクエリ + N 回の関連クエリという発行パターンそのものに由来する。ORM が SQL を隠蔽することで、開発者がクエリ発行のコストを意識しにくくなることが根本原因。

### 対策
- Rails: `includes`（自動判別）、`preload`（別クエリで先読み）、`eager_load`（LEFT OUTER JOIN）を使い分ける
- Django: `select_related`（JOIN で取得）、`prefetch_related`（別クエリで先読み）を使う
- Bullet gem（Rails）や nplusone（Django）で開発時に自動検知する
- GraphQL では DataLoader パターンを導入し、バッチでまとめて取得する
- CI に N+1 検知を組み込み、新規発生を防止する（`prosopite` gem など）

### 関連用語
- [Slow Query](#slow-query)
- [Connection Pool Exhaustion](#connection-pool-exhaustion)
- [Index の効かないクエリ](#index-の効かないクエリ)

---

## Slow Query

別名: スロークエリ / 低速クエリ

### 意味
実行に閾値以上の時間がかかるクエリ。MySQL の slow query log や PostgreSQL の `log_min_duration_statement` で検出される。単一のクエリがボトルネックとなり、DB サーバ全体の性能を引きずり下ろす。

### よくある例
- インデックスのない数百万行のテーブルに対してフルテーブルスキャンが走る
- `SELECT *` で不要な BLOB / TEXT カラムまで取得し、I/O とメモリを浪費する
- サブクエリや複雑な JOIN が最適化されず、一時テーブルの作成やファイルソートが発生する
- Rails の `where` チェーンが複雑化し、意図しない実行計画になる
- PostgreSQL の `VACUUM` が追いつかず、テーブルの膨張（bloat）によりスキャンコストが増大する

### ありがちな症状
- 特定のページや API エンドポイントだけレスポンスが異常に遅い
- `SHOW PROCESSLIST`（MySQL）や `pg_stat_activity`（PostgreSQL）で長時間実行中のクエリが見える
- CPU 使用率や IOPS が特定時刻にスパイクする
- RDS の Performance Insights でトップクエリとして常連になっている

### 近い言葉との違い
- [N+1 クエリ問題](#n1-クエリ問題): N+1 は個々のクエリは速いが総数が問題。Slow Query は1本のクエリ自体が遅い
- [Index の効かないクエリ](#index-の効かないクエリ): Slow Query の原因の一つ。インデックス未使用は slow query を生む典型的な要因
- [Connection Pool Exhaustion](#connection-pool-exhaustion): Slow Query が接続を長時間占有することでプール枯渇を引き起こす

### 対策
- `EXPLAIN` / `EXPLAIN ANALYZE` で実行計画を確認し、フルスキャンやファイルソートを特定する
- 適切なインデックスを追加する（複合インデックスの列順序にも注意）
- MySQL: `slow_query_log` を有効化し、`long_query_time` を環境に合わせて設定する（Web なら 0.1s 程度）
- PostgreSQL: `log_min_duration_statement` を設定し、`pg_stat_statements` で統計を取る
- RDS Performance Insights や Datadog Database Monitoring で継続的に監視する
- 必要に応じてクエリのリファクタリング、非正規化、マテリアライズドビューを検討する

### 関連用語
- [N+1 クエリ問題](#n1-クエリ問題)
- [Index の効かないクエリ](#index-の効かないクエリ)
- [Connection Pool Exhaustion](#connection-pool-exhaustion)

---

## Connection Pool Exhaustion

別名: 接続プール枯渇 / コネクションプール枯渇

### 意味
アプリケーションが使用する DB 接続プールの全接続が使用中となり、新しい接続を確保できなくなる状態。新規リクエストは接続待ちでタイムアウトし、アプリケーション全体が応答不能に陥る。

### よくある例
- Rails の `database.yml` で `pool: 5` のまま Puma のスレッド数を16に設定し、スレッドが接続を取り合う
- AWS Lambda + RDS の構成で同時実行数が急増し、RDS の `max_connections` に達する
- Sidekiq のワーカースレッド数とコネクションプールサイズの不整合で、バックグラウンドジョブがタイムアウトする
- 長時間トランザクションやスロークエリが接続を占有し、他のリクエストが接続を獲得できない
- Django で `CONN_MAX_AGE` を長く設定しすぎ、アイドル接続がプールを埋め尽くす

### ありがちな症状
- `ActiveRecord::ConnectionTimeoutError`（Rails）や `django.db.utils.OperationalError: connection pool exhausted` が頻発する
- 突然全リクエストがタイムアウトし、アプリケーションが固まる
- RDS の `DatabaseConnections` メトリクスが `max_connections` に張り付いている
- スケールアウト（インスタンス追加）するほど接続数が増え、逆に DB が死ぬ

### 近い言葉との違い
- [Slow Query](#slow-query): Slow Query が接続を長時間占有することで、間接的にプール枯渇を引き起こす
- [N+1 クエリ問題](#n1-クエリ問題): 大量のクエリ発行が接続の使用時間を延ばし、枯渇のリスクを高める
- [Replication Lag](#replication-lag): リードレプリカへの接続分散で接続数を削減できるが、ラグとのトレードオフが生じる

### 対策
- Rails: `pool` をスレッド数以上に設定する（Puma のスレッド数 + Sidekiq のワーカー数を考慮）
- PgBouncer / ProxySQL などのコネクションプーリングプロキシを導入し、接続を多重化する
- AWS Lambda + RDS では RDS Proxy を使い、接続の再利用とスロットリングを行う
- アイドル接続のタイムアウトを適切に設定する（`idle_timeout`、`reaping_frequency`）
- 長時間トランザクションを排除し、接続の占有時間を短くする
- `max_connections` の監視アラートを設定し、閾値（例: 80%）で通知する

### 関連用語
- [Slow Query](#slow-query)
- [N+1 クエリ問題](#n1-クエリ問題)
- [Replication Lag](#replication-lag)

---

## Phantom Read / Dirty Read / Non-repeatable Read

別名: ファントムリード / ダーティリード / 非再現リード

### 意味
DB のトランザクション分離レベルに関連する3つの異常現象の総称。トランザクション間のデータ可視性に起因し、同時実行されるトランザクション間で不整合なデータを読み取ってしまう。

- **Dirty Read**: 他のトランザクションがまだコミットしていないデータを読み取る。ロールバックされたデータを基に処理を進めてしまうリスクがある
- **Non-repeatable Read**: 同一トランザクション内で同じ行を2回読むと、その間に他トランザクションが更新・コミットしたため値が変わっている
- **Phantom Read**: 同一トランザクション内で同じ条件の SELECT を2回実行すると、その間に他トランザクションが行を挿入・削除したため結果行数が変わっている

### よくある例
- Rails のトランザクション内で在庫数を読み取り → 計算 → 更新する間に、別リクエストが在庫を変更して整合性が崩れる（Non-repeatable Read）
- 集計バッチが実行中に他のプロセスがレコードを追加し、集計結果が一貫しない（Phantom Read）
- READ UNCOMMITTED で動作するレガシーシステムが、ロールバックされた仮データを基に外部 API を呼んでしまう（Dirty Read）
- PostgreSQL のデフォルト分離レベル（READ COMMITTED）で、ループ内の各 SELECT が異なるスナップショットを見る

### ありがちな症状
- 「同じデータを見ているはずなのに集計結果が微妙にずれる」
- 在庫管理や口座残高で、あり得ないマイナス値が発生する
- バッチ処理とオンライントランザクションの同時実行でデータ不整合が起きる
- テスト環境では再現しないが、本番の高負荷時にのみ発生する

### 近い言葉との違い
- Dirty Read vs Non-repeatable Read: Dirty Read は「未コミットデータ」を読む問題、Non-repeatable Read は「コミット済みだが同トランザクション内で値が変化する」問題
- Non-repeatable Read vs Phantom Read: Non-repeatable Read は「既存行の値が変わる」、Phantom Read は「行の集合自体が変わる（増減する）」
- 分離レベルとの対応: READ UNCOMMITTED（全て発生）→ READ COMMITTED（Dirty Read を防止）→ REPEATABLE READ（Non-repeatable Read も防止）→ SERIALIZABLE（Phantom Read も防止）

### 背景・語源
ANSI SQL-92 標準でトランザクション分離レベルが定義された際に、各レベルで防げる異常現象として体系化された。Jim Gray と Andreas Reuter の著書 "Transaction Processing: Concepts and Techniques"（1993年）で理論的基盤が確立。MySQL InnoDB のデフォルトは REPEATABLE READ、PostgreSQL のデフォルトは READ COMMITTED であり、DBMS ごとに挙動が異なるため注意が必要。

### 対策
- トランザクション分離レベルを要件に応じて適切に設定する（Rails: `ActiveRecord::Base.transaction(isolation: :serializable)`）
- 在庫や残高など厳密な整合性が必要な箇所では `SELECT ... FOR UPDATE`（悲観的ロック）を使う
- PostgreSQL では REPEATABLE READ や SERIALIZABLE で `serialization_failure` エラーをリトライするパターンを実装する
- MySQL InnoDB の REPEATABLE READ ではネクストキーロックで Phantom Read を部分的に防止するが、完全ではないため要件に応じて SERIALIZABLE を検討する
- 読み取り一貫性が不要な場面（ダッシュボードの概算表示など）では低い分離レベルで性能を優先する

### 関連用語
- [Replication Lag](#replication-lag)
- [レースコンディション](concurrency.md#レースコンディション)
- [Optimistic Locking / Pessimistic Locking](concurrency.md#optimistic-locking--pessimistic-locking)

---

## Hot Spot / Hot Key

別名: ホットスポット / ホットキー / ホットパーティション

### 意味
特定のキー、パーティション、またはノードにアクセスが集中し、そのリソースがボトルネックとなる問題。分散システムにおいて負荷が均等に分散されず、一部のノードだけが過負荷になる。

### よくある例
- DynamoDB でパーティションキーに日付（`2026-04-04`）を使い、当日分のパーティションにアクセスが集中する
- Redis で特定のキー（例: 人気商品のキャッシュ、グローバルカウンター）に読み書きが殺到する
- MySQL のオートインクリメント主キーに対する INSERT が、B-Tree の右端に集中しロック競合が発生する
- ElastiCache のクラスターで特定のスロットにホットキーが偏り、1ノードだけ CPU が張り付く
- DynamoDB の WCU/RCU がパーティション単位で制限を超え、`ProvisionedThroughputExceededException` が発生する

### ありがちな症状
- 特定のパーティションやノードだけ CPU / メモリ / IOPS が高い
- DynamoDB で `ThrottlingException` が頻発するが、テーブル全体のキャパシティには余裕がある
- Redis の `slowlog` に特定キーへの操作が集中している
- スケールアウトしても特定ノードの負荷が下がらない

### 近い言葉との違い
- [Connection Pool Exhaustion](#connection-pool-exhaustion): ホットスポットは「特定リソースへの集中」、プール枯渇は「接続リソース全体の枯渇」
- [Slow Query](#slow-query): ホットスポットは個々のクエリは速くても集中が問題。Slow Query は個々のクエリ自体が遅い
- Thundering Herd: 一斉にアクセスが押し寄せるパターン。ホットスポットは「常に特定キーに偏る」パターン

### 対策
- DynamoDB: パーティションキーにランダムサフィックスを付与する（`user#1234#03` のように分散）
- DynamoDB: DAX（DynamoDB Accelerator）でリードキャッシュを挟み、ホットパーティションへの直接アクセスを減らす
- Redis: ホットキーを複数のキーに分散する（`counter:0` 〜 `counter:9` に分割し集約）
- キャッシュ層を追加し、DB への直接アクセスを減らす（CloudFront → ElastiCache → RDS）
- 書き込みが集中する場合は、バッファリング + バッチ書き込みパターン（SQS → Lambda → DynamoDB）を検討する
- 監視: DynamoDB の CloudWatch メトリクス `ConsumedReadCapacityUnits` / `ConsumedWriteCapacityUnits` をパーティション単位で追跡する

### 関連用語
- [Connection Pool Exhaustion](#connection-pool-exhaustion)
- [Replication Lag](#replication-lag)
- [Index の効かないクエリ](#index-の効かないクエリ)

---

## Index の効かないクエリ

別名: Index-Unfriendly Query / インデックス未使用クエリ

### 意味
インデックスが存在するにもかかわらず、クエリの書き方によってオプティマイザがインデックスを使用せずフルテーブルスキャンを選択してしまう問題。開発者は「インデックスを貼ったから速いはず」と思い込むが、実行計画を確認すると使われていない。

### よくある例
- `LIKE '%keyword%'`（前方一致でない LIKE）でインデックスが効かない
- `WHERE YEAR(created_at) = 2026` のようにカラムに関数を適用し、インデックスが無効化される
- 文字列カラムに数値を渡す（`WHERE phone = 09012345678`）など型の不一致で暗黙の型変換が発生する
- `WHERE col1 = 1 OR col2 = 2` で複合インデックスが使えず、フルスキャンになる
- `SELECT * FROM users WHERE age != 30` のように否定条件でインデックスが効かない
- Rails の `where.not` が意図せず全件スキャンを引き起こす
- 複合インデックス `(a, b, c)` に対して `WHERE b = 1` で先頭カラムをスキップする

### ありがちな症状
- `EXPLAIN` の結果が `Seq Scan`（PostgreSQL）や `type: ALL`（MySQL）になっている
- インデックスを追加したのにクエリが速くならない
- 本番データ量が増えるにつれて特定のクエリだけ劇的に遅くなる
- Slow Query ログの常連になっているクエリにインデックスが存在する

### 近い言葉との違い
- [Slow Query](#slow-query): インデックスの未使用は Slow Query の主要な原因の一つ
- [N+1 クエリ問題](#n1-クエリ問題): N+1 は回数の問題、インデックス未使用は1回あたりの効率の問題
- カバリングインデックス: インデックスだけで結果を返せる状態。効かないクエリの対極

### 対策
- `EXPLAIN` / `EXPLAIN ANALYZE` を習慣化し、実行計画を必ず確認する
- Rails では `ActiveRecord::Base.logger` や `explain` メソッドで開発中に確認する
- 前方一致 `LIKE 'keyword%'` ならインデックスが効く。全文検索には PostgreSQL の `pg_trgm` や Elasticsearch を使う
- カラムに関数を適用する代わりに、生成列（Generated Column）や式インデックスを使う（PostgreSQL: `CREATE INDEX ON t ((YEAR(created_at)))`）
- 複合インデックスは「左端から順に使う」原則を理解し、クエリパターンに合わせて設計する
- CI に `EXPLAIN` 結果の自動チェックを組み込む（`pg_query` でパースして Seq Scan を検知するなど）

### 関連用語
- [Slow Query](#slow-query)
- [N+1 クエリ問題](#n1-クエリ問題)
- [Migration の罠](#migration-の罠)

---

## Migration の罠

別名: DB マイグレーションの落とし穴 / Schema Migration Pitfalls

### 意味
DB スキーマのマイグレーション時に発生する、サービス停止やパフォーマンス劣化を引き起こす問題の総称。特に大規模テーブルに対する DDL 操作がロックを取得し、オンラインサービスに影響を与えるケース。

### よくある例
- MySQL の `ALTER TABLE` で数億行のテーブルにカラムを追加し、テーブル全体がロックされて数時間サービス停止する
- `NOT NULL` 制約 + `DEFAULT` 値の追加で、既存の全行にデフォルト値を書き込む更新が走る（MySQL 5.7 以前）
- Rails の `add_index` がロックを取得し、テーブルへの書き込みがブロックされる
- `rename_column` が本番で実行され、デプロイ中の旧コードが旧カラム名でエラーになる
- Django の `RunPython` マイグレーションで大量データ更新を行い、長時間トランザクションが発生する
- Terraform で RDS のパラメータグループを変更し、意図せず再起動が発生する

### ありがちな症状
- デプロイ時に DB ロック待ちでリクエストがタイムアウトする
- マイグレーション実行中に `Lock wait timeout exceeded`（MySQL）が頻発する
- ローリングデプロイで新旧コードが混在し、カラム名変更によるエラーが出る
- マイグレーションが途中で失敗し、スキーマが中途半端な状態になる

### 近い言葉との違い
- [Slow Query](#slow-query): マイグレーションは DDL（スキーマ変更）の問題、Slow Query は DML（データ操作）の問題
- [Connection Pool Exhaustion](#connection-pool-exhaustion): マイグレーションのロックが接続を長時間占有し、プール枯渇の原因になることがある
- Zero-Downtime Deployment: マイグレーションの罠を避けるための手法群

### 背景・語源
MySQL 5.6 以前は多くの `ALTER TABLE` がテーブルコピー方式で、実行中はテーブル全体がロックされていた。この問題を解決するために pt-online-schema-change（Percona）や gh-ost（GitHub）といったオンラインスキーマ変更ツールが開発された。PostgreSQL では比較的早くからオンライン DDL がサポートされていたが、`CREATE INDEX` のロックなど完全ではなく、`CONCURRENTLY` オプションが重要になる。Rails コミュニティでは `strong_migrations` gem が安全でないマイグレーションを検知する標準的なツールとなっている。

### 対策
- MySQL: pt-online-schema-change や gh-ost でオンラインスキーマ変更を行う
- PostgreSQL: `CREATE INDEX CONCURRENTLY` でロックを回避する。`add_index` には `algorithm: :concurrently` を指定
- Rails: `strong_migrations` gem を導入し、危険なマイグレーションを CI で検知する
- カラム追加は `NULL` 許容で追加 → データ埋め → `NOT NULL` 制約追加の3段階で行う
- カラム名変更はマルチステップデプロイ（新カラム追加 → 両方に書き込み → 旧カラム削除）で行う
- 大テーブルのデータマイグレーションはバッチ処理で分割実行する（`find_each` + `in_batches`）
- 本番適用前にステージング環境で同等データ量を用いてリハーサルする

### 関連用語
- [Slow Query](#slow-query)
- [Connection Pool Exhaustion](#connection-pool-exhaustion)
- [Index の効かないクエリ](#index-の効かないクエリ)
- [Replication Lag](#replication-lag)

---

## Replication Lag

別名: レプリケーション遅延 / レプリカ遅延 / Replica Lag

### 意味
プライマリ（マスター）DB とリードレプリカの間でデータの反映に遅延が生じる現象。プライマリに書き込んだデータが、レプリカに反映されるまでのタイムラグにより、レプリカから読み取ると古いデータが返される。

### よくある例
- Rails で `writing` ロールで書き込んだ直後に `reading` ロールで読み取ると、更新前のデータが返る
- ユーザーがプロフィールを更新した直後にリダイレクトされたページで、古いプロフィール情報が表示される
- RDS のリードレプリカへの大量の読み取りクエリがレプリカの CPU を圧迫し、レプリケーション処理が遅延する
- Aurora のライターインスタンスとリーダーインスタンスの間で、通常はミリ秒単位のラグが高負荷時に数秒に拡大する
- 大量の DDL やバルクインサートがプライマリで実行され、レプリカの適用が追いつかない

### ありがちな症状
- 「さっき更新したのにデータが変わっていない」というユーザーからの問い合わせ
- 書き込み直後の画面遷移で古いデータが表示され、リロードすると最新になる
- RDS の `ReplicaLag` メトリクスが増加傾向にある
- レプリカを使ったバッチ処理の結果が、プライマリのデータと一致しない

### 近い言葉との違い
- [Phantom Read / Dirty Read / Non-repeatable Read](#phantom-read--dirty-read--non-repeatable-read): 分離レベルの問題は単一 DB 内の現象。Replication Lag は複数ノード間の物理的な遅延
- Eventual Consistency: レプリケーションラグは結果整合性の具体的な発現形態
- [Connection Pool Exhaustion](#connection-pool-exhaustion): リードレプリカの活用はプール枯渇の対策になるが、ラグとのトレードオフが生じる

### 対策
- 書き込み直後の読み取りはプライマリから行う（Read-after-Write Consistency）。Rails 6+ では `connected_to(role: :writing)` で明示的に切り替える
- Rails のマルチ DB 機能で、直近の書き込みから一定時間はプライマリを読む設定を入れる（`delay` パラメータ）
- Aurora では `aurora_replica_read_consistency` パラメータで読み取り一貫性を調整できる
- レプリカのラグを CloudWatch（`ReplicaLag`）で監視し、閾値超過時にアラートを出す
- 整合性が厳密に必要な処理（決済、在庫管理）はプライマリからのみ読み取る
- レプリカへの負荷を分散するために、読み取り専用の処理を複数レプリカに分散する（Route 53 / HAProxy）
- 大量書き込み（バルクインサート、マイグレーション）はレプリカのラグを監視しながらスロットリングする

### 関連用語
- [Phantom Read / Dirty Read / Non-repeatable Read](#phantom-read--dirty-read--non-repeatable-read)
- [Connection Pool Exhaustion](#connection-pool-exhaustion)
- [Migration の罠](#migration-の罠)
- [レースコンディション](concurrency.md#レースコンディション)
