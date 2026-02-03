Rails + Vite + Docker + GitHub Actionsを前提に、効果が大きいものだけに絞って選定しています。

---

## すでにやったことのあるもの

- **Brakeman** … Rails 向け SAST。Rails アプリのソースコードを静的解析して、SQL インジェクションや XSS などのセキュリティ脆弱性を検出する専用スキャナ。
- **RuboCop** … スタイル・品質。Ruby 向けの静的コードアナライザ兼フォーマッタで、Ruby Style Guide に沿ってコードスタイルや設計上の問題をチェックし、自動整形もできる。
- **CodeQL** … Ruby / JS・TS / Actions。GitHub 製のコード解析エンジンで、Ruby や JavaScript/TypeScript などのコードをクエリ言語で解析し、脆弱性やバグを検出して code scanning アラートとして可視化できる。
- **Dependabot** … bundler + npm + actions。GitHub の依存関係アップデート機能で、bundler や npm、GitHub Actions などのライブラリやアクションの新バージョン・脆弱性を監視し、自動で PR を立てて更新提案・自動マージまで行える。

## これらと役割が被らない、追加してみたツール

### 1. **OSV Scanner**（Google）

- **役割**: 依存関係の脆弱性スキャン（OSV データベース使用）
- **対象**: Ruby（Gemfile）、JavaScript/TypeScript（package.json）など
- **特徴**: 無料・OSS。Dependabot とは別のデータソースなので、**Dependabot の補完**になる
- **導入**: ワークフロー 1 本追加するだけ。ビルド不要


### 2. **Trivy**（Aqua Security）

- **役割**: コンテナイメージの脆弱性スキャン（OS パッケージ + 言語依存関係）
- **対象**: Dockerfile からビルドしたイメージ
- **特徴**: 無料・OSS。**本番で使うイメージを push 前にスキャン**するのに向いている
- **導入**: イメージをビルドするタイミング（例: Fly デプロイ前や専用ワークフロー）で `trivy image` を実行

既に **CodeQL / Brakeman / Dependabot** で「ソース＋依存」は見ているので、**コンテナ層**を Trivy でカバーする。


### 3. **Semgrep**（Returntocorp）

- **役割**: ルールベースの SAST（バグ・脆弱性・コーディング規約）
- **対象**: Ruby, JavaScript, TypeScript など
- **特徴**: 無料枠あり。**CodeQL / Brakeman と違うルールセット**なので、追加でバグや脆弱性を拾える
- **導入**: 公式の GitHub Action をワークフローに 1 ジョブ追加

「CodeQL や Brakeman と重複するのでは？」という点は、ルールが違うので**併用**で問題ありません（アラートが増えすぎたらルールを絞る運用でよいです）。


### 4. **OSSF Scorecard**（OpenSSF）

- **役割**: リポジトリの**サプライチェーン・運用の健全性**をスコアリング（ブランチ保護、署名、依存関係の更新など）
- **対象**: リポジトリ全体（言語非依存）
- **特徴**: 無料・OSS。**設定や運用の穴**をチェックしたいときに便利
- **導入**: 公式の Scorecard Action を週次などで 1 回実行するワークフローを追加

---

## 状況に応じて検討

| ツール | 役割 | 入れるとよいケース |
|--------|------|---------------------|
| **Snyk** | 依存関係・コンテナ・IaC | 依存関係＋コンテナを 1 ツールでまとめたい、無料枠で試したい |
| **Bearer** | SAST + 秘密・PII 検出 | 秘密情報や個人データの扱いをコードでチェックしたい |
| **Dependency Review**（GitHub 標準） | PR での依存関係変更の差分チェック | Dependabot の PR をマージする前に「何が変わったか」を見たい |
| **Haskell Dockerfile Linter (hadolint)** | Dockerfile のベストプラクティス | Dockerfile の書き方をもう少し厳しくしたい |


[![Image from Gyazo](https://i.gyazo.com/327763dda3e02a94ce47558ee9b2b7dd.png)](https://gyazo.com/327763dda3e02a94ce47558ee9b2b7dd)
