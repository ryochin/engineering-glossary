# AWS / IaC

---

## ClickOps

別名: クリックオプス / コンソールポチポチ

### 意味
AWS マネジメントコンソールなどの GUI を手動操作してインフラを構築・変更すること。
IaC（Infrastructure as Code）の対極にある運用スタイルで、再現性・追跡性・レビュー性のすべてが欠落する。

### よくある例
- AWS コンソールから Security Group のルールを直接追加する
- EC2 インスタンスを手動で起動し、後から Terraform にインポートしようとして諦める
- 「とりあえず動かしたい」で ECS のタスク定義をコンソールから更新し、次のデプロイで上書きされる

### ありがちな症状
- Terraform plan で差分が出るが、誰がいつ変えたか分からない
- 本番と検証で設定が微妙に違うが、再現手順がない
- 「コンソール見れば分かる」が口癖のチームメンバーがいる

### 近い言葉との違い
- [手動変更](#手動変更): ClickOps は GUI 操作に限定、手動変更は CLI や SSH 含む広い概念
- [Drift](#drift): ClickOps は原因、Drift は結果
- [Snowflake Server](#snowflake-server): ClickOps を繰り返した結果、再現不能なサーバーになる

### 対策
- Terraform / CloudFormation で全リソースをコード管理する
- AWS コンソールへの書き込み権限を制限し、ReadOnly をデフォルトにする
- CI/CD パイプラインからのみ変更を適用する運用フローを確立する
- どうしても手動変更が必要な場合は `terraform import` と PR を必須にする

### 関連用語
- [Drift](#drift)
- [手動変更](#手動変更)
- [Snowflake Server](#snowflake-server)
- [Infrastructure as Code](#infrastructure-as-code)
- [Undocumented Resource](#undocumented-resource)

---

## Drift

別名: Configuration Drift / Terraform Drift / 構成ドリフト

### 意味
IaC で定義された「あるべき状態」と、実際のインフラの状態が乖離すること。
Configuration Drift はインフラ全般の構成ズレを指し、Terraform Drift は特に Terraform の state と実リソースの不一致を指す。どちらも根本原因は同じで、コード外の変更が入ることで発生する。

### よくある例
- 誰かがコンソールから Security Group にルールを追加し、`terraform plan` で差分が出る
- Auto Scaling で起動したインスタンスに SSH して設定を変更し、次の AMI 更新で消える
- CloudFormation スタック外で手動作成した IAM ロールがアプリから参照されている

### ありがちな症状
- `terraform plan` を実行するたびに意図しない差分が表示される
- 「plan は通るのに apply すると壊れる」と報告される
- 本番だけ挙動が違うが、コードは検証と同一

### 近い言葉との違い
- [ClickOps](#clickops): Drift の主要な原因の一つ
- [手動変更](#手動変更): Drift を引き起こす行為全般
- [Snowflake Server](#snowflake-server): Drift が蓄積した結果の状態
- [State File Locking](#state-file-locking): state の競合による Drift を防ぐ仕組み

### 対策
- `terraform plan` を CI で定期実行し、差分を Slack 等に通知する
- AWS Config Rules で構成変更を検知する
- コンソールの書き込み権限を最小化する
- Drift を検知したら即座に `terraform import` またはコード修正で解消する
- CloudFormation の Drift Detection 機能を活用する

### 関連用語
- [ClickOps](#clickops)
- [手動変更](#手動変更)
- [Infrastructure as Code](#infrastructure-as-code)
- [State File Locking](#state-file-locking)
- [Immutable Infrastructure](#immutable-infrastructure)

---

## Immutable Infrastructure

別名: イミュータブルインフラストラクチャ / 不変インフラ

### 意味
一度デプロイしたインフラを変更せず、更新が必要な場合は新しいインスタンスやコンテナを作り直す運用方針。
「修正」ではなく「置き換え」でインフラを管理することで、Drift や Snowflake Server を構造的に防ぐ。

### よくある例
- ECS / Fargate でコンテナを毎回新しいタスクとしてデプロイする
- EC2 は Packer で AMI をビルドし、新しい AMI で Auto Scaling Group を更新する
- Kubernetes の Pod は更新時に既存を terminate して新規作成する

### ありがちな症状
- サーバーに SSH して手動パッチを当てる運用が常態化している（Immutable でない兆候）
- 「本番だけこのパッチが当たっている」という状態がある
- デプロイ後に設定ファイルを手動で書き換えている

### 近い言葉との違い
- [Pet vs Cattle](#pet-vs-cattle): Immutable Infrastructure は Cattle 的運用の技術基盤
- [Snowflake Server](#snowflake-server): Immutable の対極
- [Drift](#drift): Immutable にすれば Drift が原理的に起きにくい
- Blue-Green Deployment: Immutable Infrastructure を実現するデプロイ手法の一つ

### 背景・語源
Chad Fowler が2013年のブログ記事 *Trash Your Servers and Burn Your Code* で提唱した概念。「サーバーに変更を加えるのではなく、新しいサーバーを作り直す」というアプローチ。Docker（2013年）と Terraform（2014年）の登場により実践が容易になり、クラウドネイティブ時代の標準的なインフラ運用思想となった。

### 対策
- コンテナベースのデプロイ（ECS, EKS）を採用する
- EC2 を使う場合は AMI ベースのデプロイに統一し、SSH でのログインを原則禁止する
- 設定変更は Parameter Store / Secrets Manager 経由にし、インスタンス自体は触らない
- ユーザーデータや cloud-init で初期設定を自動化する

### 関連用語
- [Pet vs Cattle](#pet-vs-cattle)
- [Snowflake Server](#snowflake-server)
- [Drift](#drift)
- [Infrastructure as Code](#infrastructure-as-code)
- [Canary Deploy](operations.md#canary-deploy)
- [Blue-Green Deployment](operations.md#blue-green-deployment)

---

## Blast Radius

別名: 爆発半径 / 影響範囲

### 意味
ある変更や障害が影響を及ぼす範囲のこと。
IaC の文脈では、1回の `terraform apply` や CloudFormation 更新で壊れる可能性のあるリソースの範囲を指す。

### よくある例
- 1つの Terraform state に VPC, RDS, ECS, IAM をすべて入れており、IAM ポリシー変更のつもりが RDS の設定も巻き込む
- CloudFormation のネストスタックが深く、親スタックの更新で子スタック全体がロールバックする
- 本番の IAM ロール変更で全マイクロサービスが認証エラーになる

### ありがちな症状
- `terraform plan` の差分が数十リソースに及ぶ
- 1つの PR のレビューに半日かかる（変更範囲が広すぎる）
- 小さな変更のつもりがサービス全体に影響する

### 近い言葉との違い
- [Terraform Module Spaghetti](#terraform-module-spaghetti): Blast Radius が大きくなる原因の一つ
- [SPOF](operations.md#spof): 障害の起点が単一であること。Blast Radius はその影響の広がり
- [Drift](#drift): Blast Radius が大きい状態で Drift が起きると被害が拡大する

### 対策
- Terraform の state を環境・サービス・レイヤーごとに分割する（例: network / compute / database）
- `terraform plan` の差分が 10 リソースを超えたら分割を検討する
- 変更は段階的にロールアウトする（Canary Deploy, Rolling Update）
- AWS Organizations + アカウント分離で障害のスコープを物理的に限定する

### 関連用語
- [Terraform Module Spaghetti](#terraform-module-spaghetti)
- [Infrastructure as Code](#infrastructure-as-code)
- [SPOF](operations.md#spof)
- [Canary Deploy](operations.md#canary-deploy)

---

## 手動変更

別名: Manual Change / Out-of-Band Change / 帯域外変更

### 意味
IaC やデプロイパイプラインを経由せず、人間が直接インフラやサーバーの設定を変更すること。
ClickOps（GUI）だけでなく、SSH でのコマンド実行や AWS CLI での直接操作も含む広い概念。

### よくある例
- 障害対応で SSH して nginx.conf を直接編集し、そのまま忘れる
- AWS CLI で `aws ec2 modify-instance-attribute` を実行し、Terraform に反映しない
- cron ジョブを手動で `/etc/cron.d/` に追加し、Ansible のプレイブックに書き忘れる

### ありがちな症状
- サーバーを再構築すると動かなくなる機能がある
- 「本番にだけ入っている設定」がドキュメントに載っていない
- `terraform plan` で「リソースを削除する」という差分が出て驚く

### 近い言葉との違い
- [ClickOps](#clickops): 手動変更のうち GUI 操作に限定したもの
- [Drift](#drift): 手動変更の結果として生じる状態
- [Snowflake Server](#snowflake-server): 手動変更が積み重なった結果
- [トライバルナレッジ](bad-practices.md#トライバルナレッジ): 手動変更の手順が口伝えでしか残っていない状態

### 対策
- 本番への変更は必ず PR 経由の IaC で行うルールを敷く
- 緊急対応で手動変更した場合、24時間以内にコード化する運用を義務づける
- AWS CloudTrail で API コールを監査し、IaC 外の変更を検知する
- SSH 接続自体を SSM Session Manager 経由に限定し、操作ログを残す

### 関連用語
- [ClickOps](#clickops)
- [Drift](#drift)
- [Snowflake Server](#snowflake-server)
- [Undocumented Resource](#undocumented-resource)
- [属人化](bad-practices.md#属人化)

---

## Snowflake Server

別名: スノーフレークサーバー / 雪の結晶サーバー

### 意味
手動変更が積み重なり、同じ構成を再現できなくなったサーバーのこと。
雪の結晶のように「世界に一つだけ」の状態になっていることから名付けられた。

### よくある例
- 3年間稼働している EC2 に歴代の担当者がそれぞれパッチを当て、構成が誰にも分からない
- 開発環境を新しく作ろうとしたが、本番の構築手順が残っておらず再現できない
- 「このサーバーだけ特別な設定が入っている」と引き継ぎ資料に書いてある

### ありがちな症状
- サーバーの再構築に数日かかる（または不可能）
- 担当者が退職すると運用が回らなくなる
- 「絶対に壊してはいけないサーバー」がある

### 近い言葉との違い
- [Pet vs Cattle](#pet-vs-cattle): Snowflake Server は Pet の極端な形
- [手動変更](#手動変更): Snowflake Server を生み出す行為
- [Drift](#drift): Snowflake Server では Drift が常態化している
- [Immutable Infrastructure](#immutable-infrastructure): Snowflake Server を構造的に防ぐアプローチ

### 背景・語源
Martin Fowler が2012年のブログ記事 *SnowflakeServer* で命名。雪の結晶のように一つとして同じものがないサーバーを指す。手動で設定を重ねた結果、再現不可能な状態になったサーバーの問題を表現。

### 対策
- 既存の Snowflake Server は段階的にコード化する（Ansible, Terraform import）
- 新規構築は必ず IaC で行い、手動変更を禁止する
- 定期的にサーバーを破棄・再構築するプラクティスを導入する（Chaos Engineering 的アプローチ）
- コンテナ化・Immutable Infrastructure への移行を計画する

### 関連用語
- [Pet vs Cattle](#pet-vs-cattle)
- [手動変更](#手動変更)
- [ClickOps](#clickops)
- [Drift](#drift)
- [Immutable Infrastructure](#immutable-infrastructure)
- [属人化](bad-practices.md#属人化)

---

## Pet vs Cattle

別名: ペット vs 家畜 / Pets and Cattle

### 意味
サーバーの扱い方を比喩で表した概念。
**Pet（ペット）**: 名前を付けて大切に育て、病気になったら治療する。障害時は復旧を試みる。
**Cattle（家畜）**: 番号で管理し、病気になったら入れ替える。障害時は新しいインスタンスに置き換える。

### よくある例
- Pet: `web-server-taro` という名前の EC2 を3年間メンテナンスし続ける
- Cattle: Auto Scaling Group で `i-0a1b2c3d` のようなインスタンスが自動的に起動・終了する
- Cattle: ECS タスクが異常終了したら自動的に新しいタスクが起動する

### ありがちな症状
- サーバーに人名や愛称が付いている（Pet の兆候）
- 「このインスタンスを止めると大変なことになる」という恐怖がある
- 障害対応の第一手が「再起動して様子を見る」ではなく「SSH して調査」

### 近い言葉との違い
- [Snowflake Server](#snowflake-server): Pet が極端になった状態
- [Immutable Infrastructure](#immutable-infrastructure): Cattle 的運用を技術的に支える基盤
- [SPOF](operations.md#spof): Pet は SPOF になりやすい

### 背景・語源
Bill Baker（Microsoft）が2012年頃に提唱したとされるメタファー。Randy Bias が2011〜2012年のプレゼンテーションで広めた。ペット（名前をつけて大切に育てる）とキャトル（番号で管理し、壊れたら交換する）というサーバー運用の対比。クラウドネイティブ時代のインフラ思想を象徴する比喩。

### 対策
- Auto Scaling Group や ECS を使い、個々のインスタンスに依存しない設計にする
- データは外部ストア（RDS, S3, EFS）に永続化し、インスタンスはステートレスにする
- サーバーに名前を付けない（タグで環境・ロールを識別する）
- 定期的にインスタンスを入れ替える仕組みを導入する

### 関連用語
- [Snowflake Server](#snowflake-server)
- [Immutable Infrastructure](#immutable-infrastructure)
- [Infrastructure as Code](#infrastructure-as-code)
- [SPOF](operations.md#spof)
- [ヒーロー運用](bad-practices.md#ヒーロー運用)

---

## Infrastructure as Code

別名: IaC / インフラのコード化

### 意味
インフラの構成をコード（Terraform, CloudFormation, Pulumi, Ansible 等）で定義・管理する手法。
バージョン管理、コードレビュー、自動テスト、CI/CD といったソフトウェア開発のプラクティスをインフラ管理に適用できる。

### よくある例
- Terraform で VPC, Subnet, Security Group, ECS クラスターを定義し、`terraform apply` でデプロイ
- CloudFormation テンプレートを CodePipeline で自動適用
- Ansible で EC2 インスタンスのパッケージ管理・設定管理を行う

### ありがちな症状
- IaC が導入されていない場合: インフラ変更のたびに手順書を書いてレビューしている
- IaC が形骸化している場合: コードはあるが `terraform apply` は手元で実行、PR レビューなし
- 「Terraform のコードと実環境が合っていない」と言われる（[Drift](#drift)）

### 近い言葉との違い
- [ClickOps](#clickops): IaC の対極
- [手動変更](#手動変更): IaC を経由しない変更
- [Immutable Infrastructure](#immutable-infrastructure): IaC は構成管理の手法、Immutable は運用方針。両者は組み合わせて使う
- Configuration Management（Ansible, Chef）: サーバー内部の構成管理。IaC はインフラリソース全体の管理

### 背景・語源
Andrew Clay Shafer と Patrick Debois が2008年の Agile Conference での議論をきっかけに DevOps ムーブメントが始まり、その中核概念として IaC が確立された。2011年の書籍 *Continuous Delivery*（Jez Humble & David Farley）や、Puppet（2005年）、Chef（2009年）、Terraform（2014年）などのツールの発展とともに普及した。Kief Morris の2016年の著書 *Infrastructure as Code* で体系化された。

### 対策
- 新規プロジェクトでは最初から Terraform / CloudFormation を導入する
- 既存環境は `terraform import` や AWS の IaC Generator で段階的にコード化する
- CI で `terraform plan` を自動実行し、PR に差分を表示する
- tfstate は S3 + DynamoDB でリモート管理し、[State File Locking](#state-file-locking) を有効にする

### 関連用語
- [ClickOps](#clickops)
- [Drift](#drift)
- [State File Locking](#state-file-locking)
- [Terraform Module Spaghetti](#terraform-module-spaghetti)
- [Blast Radius](#blast-radius)

---

## Day 2 Operations

別名: デイツーオペレーション / 運用フェーズ

### 意味
インフラやサービスの初期構築（Day 0 = 設計, Day 1 = デプロイ）後に始まる、継続的な運用・保守・改善の総称。
監視、パッチ適用、スケーリング、セキュリティ更新、コスト最適化、障害対応などが含まれる。

### よくある例
- Terraform で ECS クラスターを構築した後、コンテナの脆弱性スキャンやログ監視を設定する
- RDS のメジャーバージョンアップグレードを計画・実行する
- CloudWatch アラームのしきい値を運用実績に基づいて調整する

### ありがちな症状
- 「構築は得意だが運用が回らない」チーム
- 監視、バックアップ、パッチ適用が後回しになっている
- インフラを作ったらそれきりで、誰もメンテナンスしていない

### 近い言葉との違い
- [Toil](operations.md#toilトイル): Day 2 Operations のうち、自動化可能なのに手動で繰り返している作業
- [Runbook](operations.md#runbook): Day 2 Operations の手順を文書化したもの
- [Drift](#drift): Day 2 Operations を怠ると Drift が蓄積する

### 背景・語源
クラウドやプラットフォーム運用の文脈で使われる。Day 0 = 設計、Day 1 = 初期構築・デプロイ、Day 2 = 運用・保守・改善という分類。VMware や Red Hat のドキュメントで広く使われ、「Day 2 からが本番」という考え方を表す。初期構築（Day 1）に注力しがちだが、運用（Day 2）のコストの方が圧倒的に大きいことを強調する概念。

### 対策
- 構築時に Day 2 の運用設計（監視・アラート・バックアップ・パッチ戦略）を一緒に定義する
- Terraform コードに CloudWatch アラームや SNS トピックも含める
- AWS Systems Manager Patch Manager で OS パッチを自動化する
- 月次でコスト・セキュリティ・構成のレビューを実施する

### 関連用語
- [Infrastructure as Code](#infrastructure-as-code)
- [Drift](#drift)
- [Toil](operations.md#toilトイル)
- [Runbook](operations.md#runbook)
- [Observability](operations.md#observabilityオブザーバビリティ)

---

## State File Locking

別名: tfstate ロック / ステートロック

### 意味
Terraform の state ファイルに対する排他制御の仕組み。
複数のユーザーや CI ジョブが同時に `terraform apply` を実行した場合、state ファイルの競合や破損を防ぐためにロックを取得する。

### よくある例
- S3 + DynamoDB バックエンドで state を管理し、DynamoDB テーブルでロックを制御する
- CI パイプラインが同時に2本走り、一方が `Error: Error locking state` で失敗する
- ロックが残ったまま解除されず、後続の apply が全てブロックされる

### ありがちな症状
- `terraform apply` 実行時に `Error acquiring the state lock` が出る
- CI が突然ブロックされ、DynamoDB を手動で確認する羽目になる
- state ファイルが壊れてリソースが孤立する

### 近い言葉との違い
- [Drift](#drift): State File Locking は state の整合性を守る仕組み。Drift は state と実リソースの乖離
- [Optimistic Locking / Pessimistic Locking](concurrency.md#optimistic-locking--pessimistic-locking): 一般的なロック概念。State File Locking は Pessimistic Locking に近い
- [レースコンディション](concurrency.md#レースコンディション): ロックがない場合に state ファイルで発生する問題

### 対策
- S3 バックエンドには必ず DynamoDB テーブルを設定する（`dynamodb_table` パラメータ）
- CI/CD では同一 state への並列実行を防ぐ（ジョブの直列化 or Terraform Cloud の実行キュー）
- ロックが残った場合は `terraform force-unlock <LOCK_ID>` で解除する（原因調査は必須）
- Terraform Cloud / HCP Terraform を使えばロック管理が自動化される

### 関連用語
- [Infrastructure as Code](#infrastructure-as-code)
- [Drift](#drift)
- [レースコンディション](concurrency.md#レースコンディション)
- [デッドロック](concurrency.md#デッドロック)

---

## Terraform Module Spaghetti

別名: モジュールスパゲッティ / Terraform モジュール地獄

### 意味
Terraform モジュールの依存関係が複雑に絡み合い、変更の影響範囲が予測できなくなった状態。
モジュールの過度なネストや、モジュール間の暗黙的な依存が原因で発生する。

### よくある例
- モジュール A がモジュール B の output を参照し、B がモジュール C の output を参照し、C が A の output を参照する循環構造
- 1つのモジュールが 20 以上の変数を受け取り、何をしているか把握できない
- 社内共通モジュールが頻繁に破壊的変更を入れ、利用側が全て壊れる

### ありがちな症状
- `terraform plan` の実行に数分かかる
- 小さな変更のつもりが数十リソースの差分になる（[Blast Radius](#blast-radius) 拡大）
- モジュールのバージョンを上げるのが怖くて古いバージョンに固定したまま
- 新しいメンバーが Terraform コードを理解するのに1週間以上かかる

### 近い言葉との違い
- [Blast Radius](#blast-radius): Module Spaghetti は Blast Radius が大きくなる原因
- [Spaghetti Code](bad-practices.md#spaghetti-code): アプリケーションコードのスパゲッティ。概念は同じだが対象が異なる
- [Big Ball of Mud](anti-patterns.md#big-ball-of-mud): 構造が崩壊した状態全般。Module Spaghetti は Terraform 特有の表現
- [Accidental Complexity](anti-patterns.md#accidental-complexity): モジュール設計の失敗で生まれる不必要な複雑さ

### 対策
- モジュールは「1つの責務」に限定する（VPC モジュール、ECS モジュール、RDS モジュールなど）
- モジュールのインターフェース（variables / outputs）を最小限にする
- モジュールのバージョニングを導入し、セマンティックバージョニングで破壊的変更を明示する
- ネストは2階層まで（root → module → sub-module が限界）
- 迷ったらモジュール化せず、フラットに書く

### 関連用語
- [Blast Radius](#blast-radius)
- [Infrastructure as Code](#infrastructure-as-code)
- [Spaghetti Code](bad-practices.md#spaghetti-code)
- [Accidental Complexity](anti-patterns.md#accidental-complexity)

---

## Undocumented Resource

別名: 野良リソース / 管理外リソース / 孤立リソース

### 意味
IaC で管理されておらず、作成経緯や目的が不明なクラウドリソースのこと。
誰がいつ何のために作ったか分からない EC2, S3 バケット, IAM ロールなどが放置されている状態。

### よくある例
- 検証用に作った EC2 インスタンスが停止状態のまま半年間放置されている
- `test-bucket-20231115` という S3 バケットの用途が誰にも分からない
- 退職者が作った IAM ロールが本番アプリから参照されているが、ポリシーの意図が不明

### ありがちな症状
- AWS のコスト明細に見覚えのないリソースの料金がある
- `terraform plan` に含まれないリソースがコンソールに大量に存在する
- セキュリティ監査で「このリソースの管理者は誰か」に答えられない

### 近い言葉との違い
- [Drift](#drift): IaC 管理下のリソースが乖離する現象。Undocumented Resource はそもそも IaC 管理下にない
- [Snowflake Server](#snowflake-server): サーバー単体の再現不能性。Undocumented Resource はリソース全般の管理不在
- [手動変更](#手動変更): Undocumented Resource を生み出す行為
- [テクニカルデット](bad-practices.md#テクニカルデット): Undocumented Resource はインフラにおける技術的負債の一形態

### 対策
- AWS Config で全リソースのインベントリを取得し、Terraform state と突合する
- タグポリシーを強制し、`Owner`, `Environment`, `ManagedBy` タグがないリソースを検知する
- 定期的に未使用リソースの棚卸しを実施する（AWS Trusted Advisor, Cost Explorer）
- `terraform import` でコード管理に取り込むか、不要なら削除する

### 関連用語
- [ClickOps](#clickops)
- [手動変更](#手動変更)
- [Drift](#drift)
- [Infrastructure as Code](#infrastructure-as-code)
- [テクニカルデット](bad-practices.md#テクニカルデット)

---

## Least Privilege (IAM)

別名: 最小権限の原則 / Principle of Least Privilege / PoLP

### 意味
ユーザーやサービスに対して、業務に必要な最小限の権限のみを付与するセキュリティ原則。
AWS IAM の文脈では、IAM ポリシーのアクション・リソースを必要最小限に絞ることを指す。

### よくある例
- 悪い例: EC2 に `AdministratorAccess` を付けて「とりあえず動くようにする」
- 悪い例: Lambda に `AmazonS3FullAccess` を付けているが、実際には1つのバケットの `GetObject` しか使わない
- 良い例: ECS タスクロールに `s3:GetObject` を特定の ARN に対してのみ許可する

### ありがちな症状
- IAM ポリシーに `"Action": "*"` や `"Resource": "*"` が含まれている
- 全サービスが同じ IAM ロールを共有している
- 「権限エラーが出たら `FullAccess` を追加する」が慣習になっている
- セキュリティ監査で大量の指摘を受ける

### 近い言葉との違い
- [Blast Radius](#blast-radius): 権限が過剰だと、漏洩・誤操作時の Blast Radius が拡大する
- [ディフェンス・イン・デプス](design.md#ディフェンスインデプス): Least Privilege は多層防御の一層
- [ガードレール](design.md#ガードレール): Least Privilege は予防的ガードレールの一つ

### 対策
- IAM Access Analyzer で未使用の権限を特定し、削除する
- CloudTrail のログを基に実際に使用しているアクションだけを許可するポリシーを生成する
- サービスごとに専用の IAM ロールを作成し、共有しない
- AWS Organizations の SCP（Service Control Policy）で組織レベルの権限境界を設定する
- IAM ポリシーの `Condition` 句で IP アドレス、VPC、タグによる制限を追加する

### 関連用語
- [Blast Radius](#blast-radius)
- [Infrastructure as Code](#infrastructure-as-code)
- [ディフェンス・イン・デプス](design.md#ディフェンスインデプス)
- [ガードレール](design.md#ガードレール)
- [Undocumented Resource](#undocumented-resource)

---

## Terraform Plan/Apply の落とし穴

別名: Plan と Apply の乖離 / Phantom Diff

### 意味
`terraform plan` では差分がないように見えるのに `terraform apply` で予期しない変更が発生する、または plan で表示される差分が実際の影響と異なる現象。

### よくある例
- provider のバグで plan に表示されない変更がある
- `ignore_changes` で差分を隠しているが実リソースは乖離している
- plan は成功するが apply で API エラー（権限不足、リソース制限）

### ありがちな症状
- apply したら想定外のリソースが再作成された
- plan 上は「1 to change」だがセキュリティグループのルールが全削除→再作成
- plan を信じて apply したが本番が壊れた

### 近い言葉との違い
- [Drift](#drift): 実リソースとステートの乖離。Plan/Apply の落とし穴はツールの挙動の問題
- [Blast Radius](#blast-radius): 影響範囲。Plan/Apply の落とし穴は影響の見積もりが外れるリスク

### 対策
- CI で plan の出力を PR コメントに貼り、人間がレビューする
- `-target` で影響範囲を限定して段階的に apply する
- 本番 apply 前にステージングで同じ plan/apply を実行する

### 関連用語
- [Drift](#drift)
- [Blast Radius](#blast-radius)
- [Infrastructure as Code](#infrastructure-as-code)
- [State File Locking](#state-file-locking)

---

## タグ戦略（Tagging Strategy）

別名: リソースタグ / Tag Policy / ラベリング戦略

### 意味
AWS リソースにタグ（キーと値のペア）を体系的に付与するルール。コスト配分、環境識別、オーナーシップの特定、セキュリティポリシーの適用に使う。

### よくある例
- `Environment: production`, `Team: platform`, `CostCenter: engineering` などの標準タグセット
- AWS Organizations の Tag Policy で強制

### ありがちな症状
- リソースにタグが付いておらずコストの内訳がわからない
- タグの命名規則がバラバラ（env vs Environment vs Env）
- 誰が作ったリソースかタグから追えない

### 近い言葉との違い
- [Undocumented Resource](#undocumented-resource): タグなしリソースはドキュメント化されていないリソースの一形態
- [ClickOps](#clickops): 手動作成するとタグ付けを忘れがち
- [Infrastructure as Code](#infrastructure-as-code): IaC ではタグをコードで一元管理できる

### 対策
- 必須タグ（Environment, Team, Service, CostCenter）をチームで合意する
- Terraform の `default_tags` で全リソースに自動付与する
- AWS Config Rules でタグ未設定のリソースを検知する

### 関連用語
- [Undocumented Resource](#undocumented-resource)
- [ClickOps](#clickops)
- [Infrastructure as Code](#infrastructure-as-code)
- [Least Privilege (IAM)](#least-privilege-iam)
