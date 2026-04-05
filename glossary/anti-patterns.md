# アンチパターン

---

## Bike-shedding

別名: 自転車置き場の議論 / Parkinson's law of triviality

### 意味
些末な問題に対して不釣り合いなほど長い議論が行われる現象。重要で複雑な議題は理解が難しいため素通りされ、誰でも意見を言える簡単な話題に時間が吸われる。

### よくある例
- Terraform の命名規則やタグ付けルールで何時間も議論する一方、VPC 設計やセキュリティグループの構成はレビューされない
- Pull Request で変数名やコメントの言い回しだけ延々とコメントが付き、アーキテクチャ上の問題には誰も触れない
- CI パイプラインの Slack 通知のアイコンや色を決める会議が 1 時間続く

### ありがちな症状
- レビューコメントの大半がスタイルや命名に関するもの
- 会議で「決め」の話が出ると急に全員が発言し始める
- 重要な設計判断が「誰も反対しなかった」という理由で通っている

### 近い言葉との違い
- [Analysis Paralysis](#analysis-paralysis): 分析しすぎて動けない。Bike-shedding は分析ではなく議論に時間を使う
- [Gold Plating](#gold-plating): 不要な装飾を加える行為。Bike-shedding は議論フェーズの問題
- [Yak Shaving](#yak-shaving): 本来の目的から逸れた作業の連鎖。Bike-shedding は議論が逸れる

### 背景・語源
C. Northcote Parkinson が1957年の著書 *Parkinson's Law* で紹介した寓話に由来する。原子力発電所の建設計画を審議する委員会が、複雑な原子炉の設計には何も言えないのに、自転車置き場（bike shed）の屋根の色について何時間も議論したという話。正式名は Parkinson's law of triviality（些末の法則）。

### 対策
- linter / formatter をプロジェクト初期に導入し、スタイル議論を自動化で排除する
- レビューに「Architecture」「Style」などラベルを付け、優先度を可視化する
- 会議にタイムボックスを設け、些末な議題は非同期で処理する
- 「これは bike-shedding では？」と指摘できるチーム文化を作る

### 関連用語
- [Analysis Paralysis](#analysis-paralysis)
- [Gold Plating](#gold-plating)
- [Scope Creep](#scope-creep)

---

## Yak Shaving

別名: ヤクの毛刈り

### 意味
本来やりたかったタスクを達成するために、一見無関係な前提タスクが連鎖的に発生し、気づけば元の目的から遠く離れた作業をしている状態。

### よくある例
- Rails アプリのバグを修正しようとしたら、まずローカル環境の Ruby バージョンを上げる必要があり、そのために Homebrew を更新し、Xcode CLT を再インストールし、結局半日潰れる
- ECS のタスク定義を変更したいが、Terraform のバージョンが古くて plan が通らず、Terraform を上げたら provider も上げる必要があり、state のマイグレーションまで始まる
- Docker イメージのビルドを直すために base image を更新し、依存ライブラリの互換性問題を解き、CI のキャッシュ設定まで変更する羽目になる

### ありがちな症状
- 「あと 1 つだけやれば本題に着手できる」を何度も繰り返している
- PR の diff が当初の想定より 10 倍以上大きくなっている
- 日報に書いた作業内容が朝の計画と全く違う

### 近い言葉との違い
- [Bike-shedding](#bike-shedding): 議論が些末な方向に流れる。Yak Shaving は作業が連鎖する
- [Scope Creep](#scope-creep): 要件が徐々に膨らむ。Yak Shaving は前提条件の連鎖
- [Boiling the Ocean](#boiling-the-ocean): 最初から巨大なスコープに取り組む。Yak Shaving は意図せず膨らむ

### 背景・語源
MIT の Carlin Vieri が2000年頃に使い始めたとされる。アニメ *Ren & Stimpy* のエピソード「Yak Shaving Day」に由来し、Seth Godin が2005年のブログで広めた。「本来やりたいことをするために、一見無関係な作業を延々と連鎖的にやる羽目になる」状態を、ヤクの毛を刈るという突飛な作業に例えた。

### 対策
- 連鎖が 2 段以上になったら一度立ち止まり、元の目的を再確認する
- 前提タスクを別チケットに切り出し、ワークアラウンドで本題を先に進める
- 環境構築は Dev Container や Nix で再現性を確保し、連鎖を断つ
- 「今やるべきか？」を常に問い、yak shaving に気づいたらチームに共有する

### 関連用語
- [Bike-shedding](#bike-shedding)
- [Scope Creep](#scope-creep)
- [Boiling the Ocean](#boiling-the-ocean)
- [Accidental Complexity](#accidental-complexity)

---

## Analysis Paralysis

別名: 分析麻痺 / 過剰分析

### 意味
選択肢の比較検討や情報収集に時間をかけすぎて、意思決定や実装に着手できなくなる状態。完璧な判断を求めるあまり、何も進まない。

### よくある例
- Kubernetes か ECS か、半年間比較検討を続けて結局どちらも導入されない
- データベースを PostgreSQL にするか MySQL にするか、ベンチマーク記事を読み漁るだけで設計が始まらない
- CI ツールの選定で Jenkins / GitHub Actions / CircleCI / GitLab CI の比較表を何度も更新し続ける

### ありがちな症状
- 比較表のカラムが 20 列を超えている
- 「もう少し調べてから決めよう」が口癖になっている
- PoC すら始まっていないのに選定ドキュメントだけが充実している
- 全員が情報を持っているのに誰も決定を下さない

### 近い言葉との違い
- [Bike-shedding](#bike-shedding): 些末な議題に時間を使う。Analysis Paralysis は重要な議題でも動けない
- [Premature Optimization](#premature-optimization): 早すぎる最適化。Analysis Paralysis は最適化以前に決められない
- [Boiling the Ocean](#boiling-the-ocean): スコープが大きすぎる。Analysis Paralysis はスコープ以前に分析で止まる

### 背景・語源
古くから使われる英語の慣用表現で、特定の初出は明確でない。ソフトウェア開発では1990年代の UML/RUP 全盛期に、設計フェーズが終わらない問題として広く認知された。

### 対策
- 判断期限（タイムボックス）を事前に決める。「金曜までに決定、それ以降は覆さない」
- 決定を可逆にする設計を採用し、間違えてもやり直せるようにする（例: feature flag）
- 「最悪の選択肢でも動いている方がマシ」というマインドセットを持つ
- 小さな PoC で実際に動かしてから判断する

### 関連用語
- [Bike-shedding](#bike-shedding)
- [Premature Optimization](#premature-optimization)
- [Boiling the Ocean](#boiling-the-ocean)

---

## Premature Optimization

別名: 早すぎる最適化

### 意味
パフォーマンス上のボトルネックが計測・特定される前に、推測だけで最適化を行うこと。Donald Knuth の「Premature optimization is the root of all evil」が有名。

### よくある例
- 月間 100 リクエストの社内ツールに Redis キャッシュ層を入れ、キャッシュ整合性の問題で苦しむ
- 立ち上げたばかりの Rails アプリで N+1 クエリを恐れるあまり、全クエリを生 SQL で書く
- まだユーザーが 10 人のサービスで Kubernetes のオートスケーリングを細かくチューニングする

### ありがちな症状
- 「将来 100 万ユーザーが来たら」という仮定でインフラ設計している
- コードの可読性を犠牲にしたトリッキーな実装がある
- ベンチマークやプロファイルの結果が一度も取られていないのに「高速化」している

### 近い言葉との違い
- [Gold Plating](#gold-plating): 不要な機能を追加する。Premature Optimization は不要な性能改善
- [Analysis Paralysis](#analysis-paralysis): 分析で止まる。Premature Optimization は分析なしで動く
- [YAGNI](design.md#yagni): 不要な機能全般。Premature Optimization はその性能面の話
- [Accidental Complexity](#accidental-complexity): 結果として生まれる不要な複雑さ。Premature Optimization はその原因の一つ

### 背景・語源
Donald Knuth が1974年の論文 *Structured Programming with go to Statements* で述べた「premature optimization is the root of all evil（早すぎる最適化は諸悪の根源）」が出典。ただし Knuth 自身は Tony Hoare の言葉として引用しており、正確な原典は議論がある。

### 対策
- まず計測する。`EXPLAIN ANALYZE`、APM（New Relic / Datadog）、プロファイラを使う
- 「遅い」と感じてからが最適化のスタートライン
- 最適化する場合も、ベンチマーク結果を PR に貼る習慣をつける
- シンプルな実装 → 計測 → ボトルネック特定 → 局所最適化の順を守る

### 関連用語
- [Gold Plating](#gold-plating)
- [Accidental Complexity](#accidental-complexity)
- [Analysis Paralysis](#analysis-paralysis)

---

## Gold Plating

別名: 金メッキ / 過剰品質

### 意味
要件として求められていない機能や品質を、良かれと思って追加すること。顧客やユーザーが望んでいない「完璧さ」に時間を費やす。

### よくある例
- 管理画面に誰も使わないダッシュボードやグラフを追加する
- 社内 API にページネーション・ソート・フィルタリングを全部実装したが、クライアントは全件取得しかしない
- Terraform モジュールを汎用化しすぎて、パラメータが 50 個ある巨大モジュールになる

### ありがちな症状
- 「ついでに」「せっかくだから」「将来使うかも」が設計理由
- ドキュメント化されていない隠し機能がある
- 実装した機能の利用率を計測していない

### 近い言葉との違い
- [Premature Optimization](#premature-optimization): 不要な性能改善。Gold Plating は不要な機能追加
- [Scope Creep](#scope-creep): ステークホルダーからの要件追加。Gold Plating は開発者自身の判断
- [Feature Creep](#feature-creep): 機能が増え続ける現象。Gold Plating は 1 回の開発での過剰実装
- [NIH 症候群](#nih症候群): 外部を使わず自作する。Gold Plating は必要以上に磨く

### 対策
- ユーザーストーリーの受け入れ条件を明確にし、それ以上やらない
- 「YAGNI（You Aren't Gonna Need It）」を合言葉にする
- コードレビューで「これは今必要か？」を確認する
- MVP で出してフィードバックを得てから磨く

### 関連用語
- [Premature Optimization](#premature-optimization)
- [Feature Creep](#feature-creep)
- [Scope Creep](#scope-creep)

---

## NIH症候群

別名: Not Invented Here Syndrome / NIH シンドローム

### 意味
外部のライブラリ・サービス・ツールを採用せず、自前で再発明してしまう傾向。「自分たちで作った方が良い」という信念が合理的判断を曇らせる。

### よくある例
- 認証基盤を自前実装し、セッション管理やトークンリフレッシュのバグに悩まされる（Auth0 / Cognito を使えば解決）
- 独自のデプロイスクリプトを shell で書き続け、Capistrano や GitHub Actions で十分な処理を再発明する
- OSS の ORM を使わず独自のクエリビルダーを開発し、メンテナンスコストが膨大になる

### ありがちな症状
- 「うちのユースケースは特殊だから」が口癖
- 社内ツールのメンテナンスに専任のエンジニアが必要
- 似たような機能の社内ライブラリが複数存在する

### 近い言葉との違い
- [Gold Plating](#gold-plating): 不要な装飾。NIH 症候群は不要な再発明
- [Inner Platform Effect](#inner-platform-effect): 既存プラットフォームの再実装。NIH 症候群はそのモチベーション
- [Accidental Complexity](#accidental-complexity): 不要な複雑さ全般。NIH 症候群はその原因の一つ

### 背景・語源
「Not Invented Here」の略。自分たちが作ったものでなければ信用しない態度を指す。用語自体は製造業で古くから使われており、ソフトウェア業界では1990年代から広く使われるようになった。

### 対策
- 「Build vs Buy」の判断基準をチームで明文化する
- 自前実装のTCO（総所有コスト）を正直に見積もる。保守・ドキュメント・オンボーディングコストを含める
- まず OSS / SaaS で検証し、本当に合わない場合だけ自作する
- 自作する場合も、インターフェースは標準に合わせて後から差し替え可能にする

### 関連用語
- [Inner Platform Effect](#inner-platform-effect)
- [Gold Plating](#gold-plating)
- [Accidental Complexity](#accidental-complexity)

---

## Boiling the Ocean

別名: 海を沸かす / 壮大すぎる計画

### 意味
一度にすべてを解決しようとする、非現実的に大きなスコープでプロジェクトに取り組むこと。結果として何も完成しない。

### よくある例
- レガシーシステムの全面リプレースを 1 回のビッグバンリリースで行おうとする
- 「まず全サービスを Kubernetes に移行してから新機能開発」と計画し、移行が終わらない
- 既存の monolith を一気に 20 個のマイクロサービスに分割しようとする

### ありがちな症状
- プロジェクト計画のガントチャートが画面に収まらない
- 「フェーズ 1 が終わらないとフェーズ 2 に着手できない」構造
- 半年以上リリースがない
- 途中で要件が変わり、やり直しが発生する

### 近い言葉との違い
- [Scope Creep](#scope-creep): 徐々にスコープが膨らむ。Boiling the Ocean は最初から巨大
- [Yak Shaving](#yak-shaving): 前提タスクの連鎖で膨らむ。Boiling the Ocean は計画自体が巨大
- [Analysis Paralysis](#analysis-paralysis): 分析で止まる。Boiling the Ocean は実装に着手するが終わらない

### 対策
- Strangler Fig パターンで段階的に移行する
- 最小限のスコープで価値を届けるフェーズ分割を行う
- 「2 週間でリリースできる単位」にタスクを分解する
- 全体像は持ちつつ、各フェーズが独立して価値を持つように設計する

### 関連用語
- [Scope Creep](#scope-creep)
- [Big Ball of Mud](#big-ball-of-mud)
- [Yak Shaving](#yak-shaving)

---

## Scope Creep

別名: スコープクリープ / 要件の肥大化

### 意味
プロジェクトの進行中に、当初予定していなかった要件や機能が少しずつ追加され、スコープが際限なく膨張する現象。

### よくある例
- 「ログイン機能を作る」から始まったチケットが、パスワードリセット・二要素認証・ソーシャルログインまで膨らむ
- Terraform で VPC を構築する PR に「ついでに CloudWatch アラームも」「S3 バケットも」と追加される
- CI パイプラインの改善タスクが、テストカバレッジ計測・セキュリティスキャン・デプロイ自動化まで含むようになる

### ありがちな症状
- チケットの説明が何度も更新されている
- 見積もり時間を大幅に超過している
- 「ここまでやったならこれも」という発言が頻出する
- PR が巨大化してレビューが困難になる

### 近い言葉との違い
- [Feature Creep](#feature-creep): 機能の増殖。Scope Creep はより広く、非機能要件も含むスコープ全体の膨張
- [Gold Plating](#gold-plating): 開発者自身の判断で追加。Scope Creep はステークホルダーからの要求も含む
- [Boiling the Ocean](#boiling-the-ocean): 最初から巨大。Scope Creep は徐々に膨らむ
- [Yak Shaving](#yak-shaving): 技術的な前提の連鎖。Scope Creep はビジネス要件の追加

### 対策
- スコープを明文化し、変更管理プロセスを設ける
- 追加要件は別チケットに切り出し、バックログに積む
- PR は 1 つの目的に限定し、関係ない変更を混ぜない
- 定期的にスコープと進捗を照合し、逸脱を早期に検知する

### 関連用語
- [Feature Creep](#feature-creep)
- [Gold Plating](#gold-plating)
- [Boiling the Ocean](#boiling-the-ocean)

---

## Feature Creep

別名: フィーチャークリープ / 機能の肥大化

### 意味
プロダクトに機能が際限なく追加され続け、複雑化・肥大化する現象。個々の機能は合理的に見えても、全体としてはユーザー体験と保守性を損なう。

### よくある例
- 管理画面に要望のたびに設定項目が増え、設定画面だけで 10 ページになる
- Rails アプリの routes.rb が 500 行を超え、使われていないエンドポイントが大量にある
- systemd の unit ファイルに条件分岐が増え続け、何が起動されるか把握できない

### ありがちな症状
- 「この機能、誰が使ってるの？」に答えられない
- 新機能追加のたびに既存機能が壊れる
- オンボーディングに必要な説明が年々増えている
- テストスイートの実行時間が異常に長い

### 近い言葉との違い
- [Scope Creep](#scope-creep): プロジェクト単位のスコープ膨張。Feature Creep はプロダクト全体の機能膨張
- [Gold Plating](#gold-plating): 1 回の開発での過剰。Feature Creep は長期にわたる蓄積
- [Big Ball of Mud](#big-ball-of-mud): アーキテクチャの崩壊。Feature Creep はその原因の一つ

### 背景・語源
プロジェクト管理の分野で1990年代から広く使われている用語。明確な単一の初出はないが、PMBOK（Project Management Body of Knowledge）などで定義されている。Scope Creep はプロジェクト範囲の拡大、Feature Creep は機能の追加膨張を指す。

### 対策
- 機能の利用率を計測し、使われていない機能を定期的に廃止する
- 新機能追加時に「既存機能の廃止」をセットで検討する
- feature flag で段階的にリリースし、効果を検証してから正式採用する
- プロダクトのコアバリューを明確にし、それに合わない機能は断る

### 関連用語
- [Scope Creep](#scope-creep)
- [Gold Plating](#gold-plating)
- [Big Ball of Mud](#big-ball-of-mud)
- [Lava Flow](#lava-flow)

---

## Golden Hammer

別名: 金のハンマー / すべてが釘に見える

### 意味
1 つの技術やツールに過度に依存し、あらゆる問題をそれで解決しようとするアンチパターン。「ハンマーを持つ者にはすべてが釘に見える」が語源。

### よくある例
- あらゆるデータ保存に RDB を使い、ジョブキューもキャッシュも全部 PostgreSQL で実装する
- すべてのインフラ課題を Kubernetes で解決しようとし、静的サイトまで K8s にデプロイする
- バッチ処理もリアルタイム処理も全部 Rails の ActiveJob で処理する

### ありがちな症状
- 技術選定の議論で常に同じ結論になる
- 「このツールならなんでもできる」という発言が出る
- ツールの限界に当たっても、ワークアラウンドで無理やり対応する
- チームの採用要件が特定技術の経験のみ

### 近い言葉との違い
- [NIH 症候群](#nih症候群): 外部を拒否する。Golden Hammer は特定ツールに固執する
- [Premature Optimization](#premature-optimization): 不要な最適化。Golden Hammer は不適切なツール選択
- [Lava Flow](#lava-flow): 古い技術が残る。Golden Hammer は今の選択の問題

### 背景・語源
心理学者 Abraham Maslow の1966年の言葉「ハンマーを持つ人にはすべてが釘に見える（law of the instrument）」が起源。*AntiPatterns: Refactoring Software, Architectures, and Projects in Crisis*（1998, Brown et al.）でソフトウェアアンチパターンとして正式に定義された。

### 対策
- 技術選定時に最低 2〜3 の選択肢を比較する習慣をつける
- ADR（Architecture Decision Record）で選定理由を記録し、前提が変わったら見直す
- チーム内で異なる技術スタックの勉強会を定期開催する
- 「このツールが使えなかったらどうする？」を定期的に問う

### 関連用語
- [NIH 症候群](#nih症候群)
- [Premature Optimization](#premature-optimization)
- [Lava Flow](#lava-flow)

---

## Lava Flow

別名: 溶岩流コード / Dead Code

### 意味
かつて必要だった（あるいは必要だと思われた）コードが、目的を失った後も削除されずにコードベースに残り続ける状態。溶岩のように固まって動かせなくなる。

### よくある例
- 数年前の移行作業で作った互換レイヤーが、移行完了後も残っている
- `TODO: あとで消す` というコメントが 3 年前から放置されている
- Docker Compose に使われていないサービス定義が残っており、誰も消す勇気がない
- Rails の migration ファイルに対応するモデルがもう存在しない

### ありがちな症状
- 「このコード何してるの？」「触らない方がいい」という会話が発生する
- テストがないコードが大量にあり、削除の影響範囲が分からない
- grep すると参照がゼロだが、消すのが怖いコードがある
- デッドコードが IDE の補完候補を汚染している

### 近い言葉との違い
- [Big Ball of Mud](#big-ball-of-mud): 構造全体の崩壊。Lava Flow は不要コードの蓄積
- [Accidental Complexity](#accidental-complexity): 不要な複雑さ。Lava Flow はその一形態
- [Feature Creep](#feature-creep): 機能が増え続ける。Lava Flow は死んだ機能が残り続ける

### 背景・語源
1998年の書籍 *AntiPatterns: Refactoring Software, Architectures, and Projects in Crisis*（Brown et al.）で命名されたアンチパターン。溶岩が冷えて固まると除去が困難になるように、一度デプロイされた不要なコードが石化して誰も触れなくなる様を表現している。

### 対策
- 定期的にデッドコードを検出するツールを CI に組み込む（rubocop, eslint の no-unused-vars 等）
- コードに有効期限を設ける。`# DEPRECATED: 2025-06 までに削除` のようなコメント
- feature flag で無効化してから一定期間後に削除する運用フローを確立する
- 「消すための PR」を定期的に作るカルチャーを育てる

### 関連用語
- [Big Ball of Mud](#big-ball-of-mud)
- [Accidental Complexity](#accidental-complexity)
- [Feature Creep](#feature-creep)

---

## Big Ball of Mud

別名: 泥だんご / スパゲッティアーキテクチャ

### 意味
明確なアーキテクチャが存在せず、場当たり的な変更が積み重なって構造が崩壊したシステム。動いてはいるが、誰も全体像を把握できない。

### よくある例
- 1 つの Rails コントローラに 2000 行のビジネスロジックが詰め込まれている
- Terraform の state が 1 ファイルに全リソース入っており、plan に 30 分かかる
- systemd の unit ファイルが ExecStartPre で 10 個のスクリプトを呼び、依存関係が暗黙的

### ありがちな症状
- 新機能追加に既存機能の 3 倍の時間がかかる
- 1 箇所の変更が予想外の場所に影響する
- 「リファクタリングしたい」が常にバックログにあるが着手されない
- 新メンバーの立ち上がりに数ヶ月かかる

### 近い言葉との違い
- [Lava Flow](#lava-flow): 不要コードの蓄積。Big Ball of Mud は構造全体の問題
- [Accidental Complexity](#accidental-complexity): 不要な複雑さ。Big Ball of Mud はその極致
- [Feature Creep](#feature-creep): 機能の膨張。Big Ball of Mud はその結果としてのアーキテクチャ崩壊

### 背景・語源
Brian Foote と Joseph Yoder が1997年に発表し、1999年に正式な論文として出版した。「認識可能なアーキテクチャが存在しないシステム」を表す用語。彼らは「これが最も広く使われているソフトウェアアーキテクチャである」と皮肉を込めて述べた。

### 対策
- Strangler Fig パターンで段階的に構造を改善する
- モジュール境界を明確にし、依存方向を制御する
- Terraform なら state を分割し、モジュール化を進める
- 「触ったら少し良くする」ボーイスカウトルールをチームで実践する
- 新規コードには明確な設計方針を適用し、汚染の拡大を止める

### 関連用語
- [Lava Flow](#lava-flow)
- [Accidental Complexity](#accidental-complexity)
- [Feature Creep](#feature-creep)
- [Scope Creep](#scope-creep)

---

## Inner Platform Effect

別名: インナープラットフォーム効果

### 意味
既存のプラットフォームやフレームワークの機能を、その上にさらに汎用的な仕組みとして再実装してしまうアンチパターン。結果として劣化版の再発明が生まれる。

### よくある例
- RDB の上に独自の「スキーマレスデータストア」を構築する（EAV パターンの濫用）
- Kubernetes の上に独自のオーケストレーション層を作り、K8s の機能を再実装する
- CI/CD ツールの上に独自のワークフローエンジンを構築し、YAML で YAML を生成する仕組みになる

### ありがちな症状
- 設定ファイルがチューリング完全に近い表現力を持っている
- 「何でもできる」が売りだが、簡単なことをやるのに大量の設定が必要
- プラットフォームの学習コストがその上のアプリケーション開発より高い
- バグ報告の大半がプラットフォーム層に起因する

### 近い言葉との違い
- [NIH 症候群](#nih症候群): 外部を使わない傾向。Inner Platform Effect はその結果の一形態
- [Gold Plating](#gold-plating): 過剰な装飾。Inner Platform Effect は過剰な汎用化
- [Abstraction Inversion](#abstraction-inversion): 低レベルを高レベルで再実装。Inner Platform Effect はプラットフォーム全体の再実装

### 対策
- 汎用化は 3 つ以上の具体的なユースケースが出てから行う（Rule of Three）
- 「既存のプラットフォームの何が不足しているか」を明確にしてから拡張する
- 設定ファイルがプログラミング言語に見え始めたら危険信号
- 定期的に「この抽象化層は本当に必要か」を問い直す

### 関連用語
- [NIH 症候群](#nih症候群)
- [Abstraction Inversion](#abstraction-inversion)
- [Accidental Complexity](#accidental-complexity)
- [Gold Plating](#gold-plating)

---

## Accidental Complexity

別名: 偶発的複雑さ / 付随的複雑さ

### 意味
問題領域そのものに起因する本質的複雑さ（Essential Complexity）ではなく、技術的な選択やツール・プロセスによって人為的に生まれた不要な複雑さ。Fred Brooks の「No Silver Bullet」で提唱された概念。

### よくある例
- 3 つの異なるパッケージマネージャ（npm / yarn / pnpm）が混在し、lock ファイルの競合が日常的に発生する
- Docker Compose、Makefile、shell スクリプト、Rake タスクが同じ処理を別々に定義している
- Terraform の module が 5 段にネストされ、変数のバケツリレーが延々と続く

### ありがちな症状
- 「なぜこんなに複雑なのか」を説明できるが、「なぜこうする必要があるのか」は説明できない
- ツールのための設定ファイルがアプリケーションコードより多い
- 新しいツールを導入するたびに既存の仕組みとの整合性に苦しむ

### 近い言葉との違い
- Essential Complexity: 問題領域固有の複雑さ。Accidental Complexity は人為的な複雑さ
- [Lava Flow](#lava-flow): 不要コードの蓄積。Accidental Complexity はより広い概念
- [Big Ball of Mud](#big-ball-of-mud): 構造の崩壊。Accidental Complexity はその原因の一つ
- [Inner Platform Effect](#inner-platform-effect): 過剰な抽象化。Accidental Complexity はより一般的な概念

### 背景・語源
Fred Brooks が1986年の論文 *No Silver Bullet -- Essence and Accident in Software Engineering* で提唱した概念。ソフトウェアの複雑さを「本質的複雑さ（essential complexity）」と「偶発的複雑さ（accidental complexity）」に分類した。

### 対策
- 定期的に「この複雑さは本質的か偶発的か」を問う
- ツールやプロセスを統一し、重複を排除する
- 「動いているから触らない」ではなく、複雑さのコストを定量化する
- アーキテクチャレビューで偶発的複雑さを早期に検出する

### 関連用語
- [Lava Flow](#lava-flow)
- [Big Ball of Mud](#big-ball-of-mud)
- [Inner Platform Effect](#inner-platform-effect)
- [Premature Optimization](#premature-optimization)

---

## Resume Driven Development

別名: RDD / 履歴書駆動開発

### 意味
技術選定やアーキテクチャの判断が、プロダクトやビジネスの要件ではなく、エンジニア自身の履歴書（レジュメ）を飾るための動機で行われること。

### よくある例
- 月間 PV 1000 の社内ツールにマイクロサービス + Kubernetes + GraphQL + gRPC を導入する
- 「Rust で書き直したい」が技術的合理性ではなくキャリア戦略に基づいている
- 安定稼働している Rails アプリを「モダンだから」という理由で Next.js + Go に置き換える提案をする

### ありがちな症状
- 技術選定の理由が「面白そう」「流行っている」「経験を積みたい」
- 既存技術で十分対応できるのに新しいスタックが選ばれる
- 導入した技術のメンテナンスを提案者以外ができない
- 退職と同時にメンテナ不在になるコンポーネントがある

### 近い言葉との違い
- [Golden Hammer](#golden-hammer): 慣れた技術への固執。Resume Driven Development は新しい技術への飛びつき
- [Premature Optimization](#premature-optimization): 不要な最適化。RDD は不要な技術導入
- [NIH 症候群](#nih症候群): 外部を拒否する。RDD は新しい外部技術を過剰に採用する

### 対策
- ADR に「なぜこの技術を選んだか」をビジネス要件ベースで記録する
- 技術選定に「既存チームが運用できるか」という評価軸を入れる
- 新技術の学習は業務外のサイドプロジェクトや 20% ルールで行う
- 「この人が辞めても運用できるか」テストを適用する

### 関連用語
- [Golden Hammer](#golden-hammer)
- [Premature Optimization](#premature-optimization)
- [NIH 症候群](#nih症候群)
- [Accidental Complexity](#accidental-complexity)

---

## Abstraction Inversion

別名: 抽象化の逆転

### 意味
高レベルの抽象化の上に、その抽象化が隠蔽している低レベルの機能を再実装してしまうこと。抽象化レイヤーが邪魔になり、本来アクセスできるはずの機能を迂回して実現する。

### よくある例
- ORM の上で生 SQL を組み立てる独自ヘルパーを作り、ORM のエスケープ処理をバイパスする
- Docker の上で systemd を動かし、コンテナ内でプロセス管理を再実装する
- serverless フレームワーク（Lambda）の上で長時間バッチ処理を無理やり実行するために、再帰呼び出しの仕組みを自作する

### ありがちな症状
- フレームワークの内部実装に依存したハックが多い
- 「このフレームワークでは○○ができない」と言いつつ、そのフレームワークを使い続けている
- 抽象化レイヤーを剥がした方がシンプルになるケースが多い
- monkey patch や private API へのアクセスが常態化している

### 近い言葉との違い
- [Inner Platform Effect](#inner-platform-effect): プラットフォーム全体の再実装。Abstraction Inversion は特定の低レベル機能の再実装
- [Accidental Complexity](#accidental-complexity): 不要な複雑さ全般。Abstraction Inversion はその特定パターン
- [NIH 症候群](#nih症候群): 外部を使わない。Abstraction Inversion は外部を使った上でその内部を再実装

### 対策
- 抽象化レイヤーが合わないなら、レイヤーごと変更する決断をする
- 「この抽象化の下に潜る必要があるか？」を設計レビューで問う
- フレームワークの想定する使い方に沿えないなら、別のフレームワークを検討する
- 例: ORM が合わないクエリは、ORM を捨てるのではなく、その部分だけ生 SQL を公式な方法で使う

### 関連用語
- [Inner Platform Effect](#inner-platform-effect)
- [Accidental Complexity](#accidental-complexity)
- [NIH 症候群](#nih症候群)
- [Golden Hammer](#golden-hammer)

---

## Second System Effect

別名: セカンドシステム効果 / 第二システム症候群

### 意味
最初に成功したシンプルなシステムの後継として設計する2番目のシステムが、機能過多・過度な設計により失敗するアンチパターン。

### よくある例
- 成功した v1 のマイクロサービスをフルリライトして「全部入り」の v2 を作ろうとして頓挫する
- 小さな CLI ツールが成功した後に GUI + プラグインシステム + マーケットプレイスを持つ v2 を企画する

### ありがちな症状
- リライトプロジェクトが当初見積もりの3倍以上かかっている
- v1 で後回しにした機能を全部 v2 に詰め込もうとしている
- v1 を維持しながら v2 を開発する二重運用が長期化

### 近い言葉との違い
- [Gold Plating](#gold-plating): 不要な装飾の追加。Second System Effect はシステム全体の過剰設計
- [Scope Creep](#scope-creep): スコープの漸進的拡大。Second System Effect は最初から巨大なスコープで始まる
- [Big Ball of Mud](#big-ball-of-mud): アーキテクチャの崩壊。Second System Effect は「きれいに作り直そう」として失敗する

### 背景・語源
Fred Brooks が *The Mythical Man-Month*（1975）で提唱。IBM System/360 の後継 OS/360 の開発での経験に基づく。「建築家が最初の家はシンプルに建てるが、2番目の家では我慢していたすべてのアイデアを詰め込もうとする」と表現した。

### 対策
- Strangler Fig パターンで段階的に移行する
- リライトではなくリファクタリングを優先する
- v2 のスコープを v1 と同等に絞り、改善は段階的に行う

### 関連用語
- [Gold Plating](#gold-plating)
- [Scope Creep](#scope-creep)
- [Big Ball of Mud](#big-ball-of-mud)

---

## Cargo Cult Agile

別名: なんちゃってアジャイル / Agile Theater

### 意味
アジャイル開発の形式（スタンドアップ・スプリント・カンバンボード等）だけを真似て、その背後にある原則や価値観を理解・実践していない状態。

### よくある例
- デイリースタンドアップをやっているが単なる進捗報告会になっている
- スプリントレトロスペクティブで出た改善アクションが毎回放置される
- 2週間スプリントだが実質ウォーターフォールのミニ版

### ありがちな症状
- 「アジャイルやってます」と言うが顧客フィードバックが入る仕組みがない
- チームに自律性がなくすべてマネージャーが決定する
- 「アジャイルだから仕様書はいらない」と言って何もドキュメントがない

### 近い言葉との違い
- [Cargo Cult Programming](bad-practices.md#cargo-cult-programming): コードレベルの理解なき模倣。Cargo Cult Agile はプロセスレベルの模倣
- [Bike-shedding](#bike-shedding): 些末な議論に時間を使う。セレモニーの形式にこだわるのは Cargo Cult Agile 的

### 背景・語源
Cargo Cult Programming（Richard Feynman のカーゴ・カルト・サイエンス）の概念をアジャイル開発に適用したもの。アジャイルマニフェスト（2001）の普及とともに、形式だけを導入する組織が増えたことへの批判として2010年代に広まった。

### 対策
- アジャイルマニフェストの4つの価値と12の原則に立ち返る
- 「このセレモニーの目的は何か？」を定期的に問い直す
- 外部のアジャイルコーチを招いてプロセスを客観視する

### 関連用語
- [Cargo Cult Programming](bad-practices.md#cargo-cult-programming)
- [Bike-shedding](#bike-shedding)

---

## Shiny Object Syndrome

別名: 新しもの好き症候群 / Hype Driven Development

### 意味
新しい技術・ツール・フレームワークに飛びつき、既存の技術スタックを十分に活用する前に乗り換えようとするアンチパターン。

### よくある例
- 「Kubernetes が流行っているから」で Docker Compose で十分なのに移行を始める
- 毎年フロントエンドフレームワークを乗り換える
- 新しい DB が出るたびに PoC を始めるが本番投入しない

### ありがちな症状
- 技術スタックが統一されておらず、サービスごとに異なるフレームワークを使っている
- 新技術の PoC ばかりで既存システムの改善が後回し
- 採用した技術のドキュメントが古いバージョンのまま

### 近い言葉との違い
- [Resume Driven Development](#resume-driven-development): 個人のキャリアのために技術を選ぶ。Shiny Object は組織全体の傾向
- [NIH 症候群](#nih症候群): 外部のものを信用しない。Shiny Object は逆に外部の新しいものに飛びつく
- [Golden Hammer](#golden-hammer): 特定技術への固執。Shiny Object はその逆で固執せずに次々変える

### 対策
- 技術選定に ADR（Architecture Decision Record）を導入し、選定理由を記録する
- 「この技術は2年後もメンテナンスされているか？」を判断基準にする
- Technology Radar で技術の成熟度を評価してから採用する

### 関連用語
- [Resume Driven Development](#resume-driven-development)
- [NIH 症候群](#nih症候群)
- [Golden Hammer](#golden-hammer)
