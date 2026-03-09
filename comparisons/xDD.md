###「xDD（Development Methodologies）完全版リスト」


# xDD (Driven Development) 手法完全リスト

ソフトウェア開発における「何を原動力（Drive）にするか」による開発手法の分類一覧です。
エンジニアリングの基本原則から、AI時代の最新トレンド、現場の実情を表すスラングまでを網羅しています。

## 1. 品質・テスティング中心 (Quality & Testing)
コードの正当性と品質を担保するための、エンジニアリングの基礎となる手法群。

* **TDD (Test-Driven Development): テスト駆動開発**
    * 「Red-Green-Refactor」サイクルを回し、テストを先に書いてから実装を行う手法。
* **BDD (Behavior Driven Development): ビヘイビア駆動開発**
    * TDDの発展形。「Given-When-Then」形式の自然言語で振る舞いを記述し、ビジネス側との共通認識を作る。
* **ATDD (Acceptance Test Driven Development): 受け入れテスト駆動開発**
    * 開発着手前に「ユーザーの受け入れ条件」を定義し、それを満たすことをゴールに開発する。

## 2. 設計・構造・モデリング中心 (Design & Structure)
システムの複雑性やコンポーネント構成を整理し、堅牢な設計を導く手法群。

* **DDD (Domain-Driven Design): ドメイン駆動設計**
    * 業務領域（ドメイン）の専門家と連携し、ビジネス概念をモデルとしてコードに落とし込む。複雑なシステムに有効。
* **CDD (Component-Driven Development): コンポーネント駆動開発**
    * 画面全体ではなく、ボタンやリストなどの部品（UIコンポーネント）単位で開発・カタログ化（Storybook等）し、ボトムアップで構築する。
* **MDD (Model-Driven Development): モデル駆動開発**
    * UMLなどのモデル図を設計の正とし、そこからコードを自動生成する手法。
* **FDD (Feature-Driven Development): 機能駆動開発**
    * 「ユーザーにとって価値ある機能」単位でリスト化し、短いイテレーションで反復開発する。

## 3. プロセス・運用・自動化中心 (Process & Ops)
チームのワークフロー、タスク管理、自動化パイプラインを主軸にした手法群。

* **TiDD (Ticket-Driven Development): チケット駆動開発**
    * Jira/Redmine等の課題管理システムと連携。「チケットのないコミットは禁止」とし、タスクの透明性を担保する。
* **GADD (GitHub Actions Driven Development): CI/CDパイプライン駆動**
    * コードを書く前にCI/CD（自動テスト・デプロイ）環境を整備し、パイプラインが通ることを確認しながら開発を進めるDevOps重視の手法。
* **RDD (Requirement-Driven Development): 要件駆動開発**
    * 要件定義書を正とし、その仕様を満たすことを最優先に進める伝統的アプローチ（ウォーターフォールに近い）。

## 4. ドキュメント・概念設計中心 (Documentation & Concept)
「何をどう作るか」の言語化・文書化を先行させる手法群。

* **RDD (Readme-Driven Development): Readme駆動開発**
    * 実装前にユーザー向けマニュアル（Readme）を執筆する。APIやCLIツール開発において「使いやすさ（DX）」を最初に定義できるため有効。
* **SDD (Specification-Driven Development): 仕様駆動開発 (AI Era)**
    * 自然言語の仕様書やプロンプトを詳細に定義し、AIツール（Spec Kit等）を用いて実装の大半を生成させる次世代手法。

## 5. AI・エージェント中心 (AI & Automation)
人間ではなくAIが主体となって開発サイクルを回す未来型手法。

* **AIDD (AI-Driven Development): AI駆動開発**
    * DevinやCursor等のAIエージェントが、計画・コーディング・デバッグを自律的あるいは半自律的に主導する。

---

## 6. 現場のリアル・アンチパターン (Anti-Patterns & Jokes)
開発現場の「あるある」や「悲哀」を風刺した俗語。実務で頻出するが推奨はされない。

| 名称 | 正式名称 (ジョーク) | 概要 |
| :--- | :--- | :--- |
| **GGTD / SO-DD** | **Google / Stack Overflow Driven** | エラーが出たら即検索＆コピペ。現代開発者の基本動作（近年はChatGPT Drivenへ移行中）。 |
| **DDD (裏)** | **Deadline-Driven Development** | **締め切り駆動。**「品質や設計より納期厳守」。デスマーチの主要因。 |
| **MADD** | **Meeting-Driven Development** | **会議駆動。** 実装時間より会議時間が長く、決定事項が二転三転して進まない状態。 |
| **HDD** | **Hype-Driven Development** | **ハイプ（流行）駆動。** 課題解決のためではなく「流行っている技術を使いたい」だけで技術選定する。 |
| **LADD** | **Last-Minute Driven Development** | **直前駆動。** リリース直前まで本気を出さず、最後に徹夜で帳尻を合わせる（夏休みの宿題方式）。 |
| **YAGNI-DD** | **You Ain't Gonna Need It** | **「それ要らんやろ」駆動。** 過剰機能を徹底的に削ぎ落とす（※これは本来良いプラクティスだが、極端なミニマリズムを指す場合もある）。 |