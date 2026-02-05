# Rails 8.1.2 の標準セキュリティ機能

Rails 8.1.2は「セキュア・バイ・デフォルト」の設計思想に基づき、Webアプリケーションの主要な脆弱性に対する包括的な保護機能を標準で提供しています [^10][^7]。

## CSRF（クロスサイトリクエストフォージェリ）対策

Railsは認証済みユーザーのセッションを悪用した不正リクエストを防ぐため、CSRFトークンによる保護をデフォルトで有効化しています [^11][^16]。

### 主な機能

- **自動トークン生成**: フォームヘルパーが自動的にCSRFトークンを挿入
- **protect_from_forgery**: `ApplicationController`でデフォルト有効
- **Turbo/Ajax対応**: `X-CSRF-Token`ヘッダーによるリクエスト検証

```ruby
class ApplicationController < ActionController::Base
  # デフォルトで有効、カスタマイズも可能
  protect_from_forgery with: :exception
  
  # APIモードでは異なる戦略を選択可能
  protect_from_forgery with: :null_session, if: -> { request.format.json? }
end
```

ビューでは`csrf_meta_tags`ヘルパーを使用してメタタグを生成します [^16]：

```erb
<head>
  <%= csrf_meta_tags %>
</head>
```

## XSS（クロスサイトスクリプティング）対策

Railsは出力時の自動エスケープとサニタイズ機能により、悪意のあるスクリプト注入を防止します [^11][^8]。

### 自動エスケープ

ERBテンプレートでの出力は自動的にHTMLエスケープされます：

```erb
<!-- 自動的にエスケープされる -->
<%= @user.bio %>

<!-- HTMLとして出力したい場合は明示的に指定 -->
<%= @user.bio.html_safe %>
```

### サニタイズヘルパー

許可リスト方式で安全なHTMLタグのみを許可できます [^8]：

```ruby
tags = %w(a acronym b strong i em li ul ol h1 h2 h3 h4 h5 h6 blockquote br cite sub sup ins p)
sanitize(user_input, tags: tags, attributes: %w(href title))
```

## SQLインジェクション対策

Active Recordはプリペアドステートメントを自動的に使用し、SQLインジェクション攻撃を防止します [^11][^1]。

### 安全なクエリの書き方

```ruby
# 安全: パラメータ化されたクエリ（推奨）
User.where(email: params[:email])
User.where("email = ? AND status = ?", params[:email], params[:status])
User.where("name = :name", name: params[:name])

# 危険: 直接的な文字列展開（警告が出る）
User.where("email = '#{params[:email]}'")  # 絶対に避ける
```

外部入力を直接SQLフラグメントに渡す場合は、必ず位置指定ハンドラまたは名前付きハンドラを使用してください [^1]。

## 認証機能ジェネレータ（Rails 8新機能）

Rails 8.0から組み込みの認証機能ジェネレータが追加され、外部gemなしでセキュアな認証システムを構築できます [^12][^13]。

```bash
rails generate authentication
```

### 生成される機能

| コンポーネント | 説明 |
|---------------|------|
| Userモデル | `has_secure_password`によるbcryptハッシュ化 |
| Sessionモデル | IPアドレス・User Agent追跡 |
| SessionsController | ログイン/ログアウト処理 |
| PasswordsController | パスワードリセット機能 |
| PasswordsMailer | リセットメール送信 |

### セキュリティ特徴

- **bcryptハッシュ化**: パスワードは不可逆的にハッシュ化されて保存 [^13]
- **認証メソッド**: `User.authenticate_by`による安全な認証
- **セッション追跡**: IPアドレスとUser Agentを記録

```ruby
class User < ApplicationRecord
  has_secure_password
  has_many :sessions, dependent: :destroy
  normalizes :email_address, with: -> e { e.strip.downcase }
end
```

## レート制限API（Rails 8新機能）

Rails 8.0で導入されたビルトインレート制限は、ブルートフォース攻撃や資格情報スタッフィング攻撃を防止します [^26][^29][^7]。

### 基本的な使用方法

```ruby
class SessionsController < ApplicationController
  # IPアドレスベースで3分間に10回まで
  rate_limit to: 10, within: 3.minutes, only: :create
end
```

### カスタマイズオプション

```ruby
class SignupsController < ApplicationController
  rate_limit to: 50, within: 1.minute,
    by: -> { request.domain },  # カスタム識別子
    with: -> { redirect_to root_path, alert: "Too many requests!" },  # カスタムレスポンス
    only: :create
end
```

制限を超えたリクエストには、デフォルトで`429 Too Many Requests`が返されます [^26][^29]。

## セッション管理

Railsはセッションハイジャックやセッション固定攻撃に対する複数の保護機能を提供します [^6]。

### CookieStoreの暗号化

- **デフォルト暗号化**: セッションデータは`secret_key_base`から導出された鍵で暗号化
- **署名付きcookie**: 改ざん検知機能

```ruby
# config/credentials.yml.enc（復号後）
secret_key_base: 492f...
```

### セッション固定攻撃対策

ログイン成功後に新しいセッションを発行することで対策できます：

```ruby
def create
  if user = User.authenticate_by(params.permit(:email_address, :password))
    reset_session  # セッション固定攻撃対策
    start_new_session_for user
    redirect_to after_authentication_url
  end
end
```

### SSL強制

```ruby
# config/application.rb
config.force_ssl = true
```

## Strong Parameters（マスアサインメント対策）

Rails 4以降標準となったStrong Parametersは、意図しない属性の更新を防止します [^18][^24][^22]。

```ruby
class UsersController < ApplicationController
  def create
    @user = User.new(user_params)
    # ...
  end

  private

  def user_params
    # 明示的に許可された属性のみ受け入れる
    params.require(:user).permit(:name, :email, :password)
    # adminなど未許可の属性は自動的に除外される
  end
end
```

## Active Record暗号化（Rails 7以降）

データベースに保存される機密データをアプリケーションレベルで暗号化できます [^17][^20]。

### 設定

```bash
bin/rails db:encryption:init
```

### 使用方法

```ruby
class User < ApplicationRecord
  encrypts :email, deterministic: true  # 検索可能な暗号化
  encrypts :ssn  # ランダム暗号化
end
```

暗号化されたデータはJSON形式で保存され、暗号文はBase64エンコードされます [^17]。

## Content Security Policy（CSP）

XSS攻撃をさらに防止するため、CSPヘッダーを設定できます [^27][^10]。

```ruby
# config/initializers/content_security_policy.rb
Rails.application.configure do
  config.content_security_policy do |policy|
    policy.default_src :self
    policy.script_src :self, :https, :nonce  # nonce対応
    policy.style_src :self, :https
    policy.img_src :self, :https, :data
    policy.object_src :none
  end
end
```

### Nonceの使用

インラインスクリプトを安全に実行するためのnonce機能を提供：

```erb
<head>
  <%= csp_meta_tag %>
</head>

<script nonce="<%= content_security_policy_nonce %>">
  // 安全なインラインスクリプト
</script>
```

## HTTPセキュリティヘッダー

Railsはデフォルトで以下のセキュリティヘッダーを設定します：

| ヘッダー | 値 | 目的 |
|---------|-----|------|
| X-Frame-Options | SAMEORIGIN | クリックジャッキング対策 |
| X-Content-Type-Options | nosniff | MIMEタイプスニッフィング防止 |
| X-XSS-Protection | 1; mode=block | ブラウザXSSフィルター有効化 |
| X-Permitted-Cross-Domain-Policies | none | Flash/PDFクロスドメイン防止 |
| Referrer-Policy | strict-origin-when-cross-origin | リファラー情報の制御 |

## Brakemanセキュリティスキャナ

Rails 8からBrakemanがデフォルトでGemfileに追加され、静的解析によるセキュリティ脆弱性の検出が可能です [^34][^31][^28]。

### 検出できる脆弱性

- SQLインジェクション
- クロスサイトスクリプティング（XSS）
- リモートコード実行（RCE）
- セッション操作
- DoS脆弱性
- コマンドインジェクション

### 実行方法

```bash
bundle exec brakeman
```

### 出力例

```
== Brakeman Report ==
Application Path: /path/to/app
Rails Version: 8.1.2
Brakeman Version: 7.0.2
Security Warnings: 0
```

Brakemanは偽陽性を出すこともあるため、ignoreリストで管理する運用が推奨されます [^1][^28]。

## ログフィルタリング

機密情報がログに出力されないよう、デフォルトでパラメータフィルタリングが設定されています：

```ruby
# config/initializers/filter_parameter_logging.rb
Rails.application.config.filter_parameters += [
  :passw, :secret, :token, :_key, :crypt, :salt, :certificate, :otp, :ssn
]
```

フィルタされたパラメータは`[FILTERED]`として出力されます。

## その他のセキュリティ機能

### Host Authorization

許可されたホストからのリクエストのみを受け入れます：

```ruby
# config/application.rb
Rails.application.config.hosts << "example.com"
```

### Permissions Policy

ブラウザ機能へのアクセスを制御できます：

```ruby
Rails.application.config.permissions_policy do |policy|
  policy.camera :none
  policy.microphone :none
  policy.geolocation :none
end
```

## セキュリティベストプラクティス

1. **許可リスト方式を徹底**: 禁止リストではなく許可リストでフィルタリング
2. **ユーザー権限チェック**: クエリにユーザーのアクセス権を含める
   ```ruby
   @project = current_user.projects.find(params[:id])
   ```
3. **正規表現の注意**: Rubyでは`^`と`$`ではなく`\A`と`\z`を使用
   ```ruby
   /\Ahttps?:\/\/[^\n]+\z/i  # 安全
   /^https?:\/\/[^\n]+$/i    # 危険
   ```
4. **定期的なセキュリティアップデート**: Rails 8では2年間のセキュリティフィックスが提供される [^7]

## まとめ

Rails 8.1.2は以下のセキュリティ機能を標準で備えています：

| カテゴリ | 機能 |
|---------|------|
| 認証・認可 | 認証ジェネレータ、Strong Parameters |
| リクエスト保護 | CSRF対策、レート制限 |
| インジェクション対策 | SQLインジェクション、XSS自動エスケープ |
| セッション | 暗号化Cookie、セッション固定対策 |
| データ保護 | Active Record暗号化、ログフィルタリング |
| ヘッダー | CSP、HTTPセキュリティヘッダー |
| 静的解析 | Brakeman |

これらの機能により、開発者はセキュリティを意識しながらも、効率的にアプリケーションを構築できます。

---

## References

1. [Kaigi on Rails 2024に参加してきました！ - あたまがきんに君](https://note.aiken-to-osanpo.dev/n/n10260c24a15e) - デプロイ時に長時間非同期処理が残ってしまっている場合に今までは指差し確認を行なっていたが、ジョブの中断・再開処理を実装して解決したという発表です ...

6. [Rails 8.1 beta: Key security features for employers](https://www.linkedin.com/posts/sorcersawan_railssecurity-applicationsecurity-rubyonrails-activity-7330865218740191232-lD5E) - The anticipated Rails 8.1 beta brings several features that forward-thinking employers are asking ab...

7. [The State of Security in Rails 8](https://www.rubyevents.org/talks/the-state-of-security-in-rails-8) - - Rate Limiting Feature: Rails now supports built-in rate limiting to guard against credential stuff...

8. [Rails セキュリティガイド](https://railsguides.jp/v8.0/security.html) - XSSやインジェクションによる攻撃を防ぐために、アプリケーションのレスポンスヘッダーに Content-Security-Policy （CSP）を定義することが推奨されています。Railsでは、 ....

10. [Unlock Faster Web Development: Rails 8 Still Leads](https://reinteractive.com/articles/upgrades-and-development/rails-8-modern-web-framework) - Security is paramount in Rails 8, with built-in features and supporting gems that minimise vulnerabi...

11. [【2025年版】Rails 8とLaravel 12のセキュリティ実装比較](https://enjoydarts.blog/archives/767) - XSS対策の実装 Rails. CSRF対策の実装 Rails 8 Rails 8では、CSRF対策がデフォルトで有効化されており、 ApplicationController で自動的に適用されます...

12. [Rails 8 & Rails 8.1: features to speed up your release cycle](https://rubyroidlabs.com/blog/2025/11/rails-8-8-1-new-features/) - A complete testing suite. A powerful system for talking to your database. A way to send emails. A se...

13. [Built-in Authentication in Rails 8.0 – A Technical Deep Dive ...](https://andriifurmanets.com/blogs/built-in-authentication-in-rails) - Rails 8.0 includes an authentication generator that scaffolds a basic email/password auth setup. The...

16. [Rails CSRF Protection: Best Practices Checklist - USEO](https://useo.tech/ruby-tech/rails-csrf-protection-best-practices-checklist/) - Rails comes with built-in CSRF (Cross-Site Request Forgery) protection, but you can further customis...

17. [Active Record Encryption](https://guides.rubyonrails.org/active_record_encryption.html) - Active Record supports application-level encryption by allowing you to declare which attributes shou...

18. [ActionController::StrongParameters - Rails API](https://api.rubyonrails.org/v7.2/classes/ActionController/StrongParameters.html) - It provides an interface for protecting attributes from end-user assignment. This makes Action Contr...

20. [Active Record Encryptionを使って属性値を暗号化するメモ](https://madogiwa0124.hatenablog.com/entry/2022/02/20/153314) - Active Record Encryptionは、Rails 7から導入されたActive Recordの新機能です。 特定の属性値をシンプルなDSLで暗号化して扱うことができます。

22. [A Complete Guide to Ruby on Rails Security Measures 🛡️](https://railsdrop.com/2025/05/11/a-complete-guide-to-ruby-on-rails-security-measures/) - In this post, we'll walk through essential Rails security measures, tackle real-world threats, and s...

24. [Deep Dive Into Rails ActionController Strong Parameters](https://blog.saeloun.com/2025/02/18/deep-dive-into-rails-action-controller-strong-parameters/) - Strong parameters allow us to explicitly permit and require specific attributes in the controller, p...

26. [Rails 8.0 built-in Rate Limiting](https://global.moneyforward-dev.jp/2024/12/17/rails-8-0-built-in-rate-limiting/) - How It Works. Rails built-in Rate Limiting uses the application's caching layer to track request cou...

27. [Configuring CSP Nonce in Ruby on Rails 7/8](https://railsdrop.com/2025/09/27/configuring-csp-nonce-in-ruby-on-rails-7-8/) - Content Security Policy (CSP) adds a powerful security layer to prevent Cross Site Scripting (XSS) a...

28. [Preparing for Rails 8? Learn Brakeman, the Built-in ...](https://www.youtube.com/watch?v=MXaR-RV35sI) - In this video, we'll dive into Brakeman (available at https://brakemanscanner.org/), a powerful stat...

29. [Rails 8 introduces a built-in rate limiting API](https://www.bigbinary.com/blog/rails-8-rate-limiting-api) - Rails 8.0 brings a native rate-limiting feature to the Action Controller, streamlining the process a...

31. [Adding SimpleCov & Brakeman To Our Application For CI ...](https://railsdrop.com/2025/05/05/rails-8-setup-simplecov-brakeman-for-test-coverage-security/) - In this post, we'll walk through integrating two powerful tools into your Rails 8 app: SimpleCov: fo...

34. [Rails 8.0 adds Brakeman](https://www.shakacode.com/blog/rails-8-adds-brakeman-by-default/) - Rails 8 adds Brakeman by default to new apps. Brakeman acts as a security shield for your Rails proj...
