# AI Development Context: "Saikyo" Minimal-Friction Infrastructure

## 1. Core Philosophy (System over Goals)
このプロジェクトでは「目標（Goal）」ではなく、開発の摩擦を最小化し、自動的に継続できる「仕組み（System）」を重視する。
- **Zero Cost:** 可能な限り無料枠（Free Tier）で完結させる。
- **Zero Friction:** デプロイ、DB接続、認証の構築に時間をかけない。
- **Graceful Degradation:** 制限到達時に「ただ動かなくなる」のではなく、ローカルモードへの切り替えやエラーハンドリングを適切に行う「哀愁のある堅牢さ」を目指す。

## 2. Standard Tech Stack (The "Saikyo" Selection)
AIはコード生成や提案を行う際、以下のサービスを第一選択とすること。

### [Frontend & Hosting]
- **Cloudflare Pages:** 転送量（Bandwidth）が無制限のため、Web公開のデフォルトとする。
- **Vercel:** Next.jsを高度に活用する場合のみ許容。

### [Backend & Database]
- **Supabase (Postgres):** メインDB、Auth、Storage、Realtime通信の統合基盤。
- **Cloudflare Workers:** 軽量なAPI、Webhook、エッジ処理用。
- **Upstash (Redis):** レート制限、キャッシュ、一時的な状態管理。

### [Real-time / P2P]
- **Supabase Realtime:** DB変更の購読。
- **LiveKit Cloud:** 低遅延な通信が必要なゲームやメディア処理。

## 3. Implementation Rules for AI
AIがコードを書く際の制約事項：

1. **Environment Variables:** 接続情報は必ず `.env` を使用し、Supabase等のクライアント初期化コードを定型化すること。
2. **Limit Handling:** APIリクエストには必ず `try-catch` を入れ、制限（429等）に達した際のフォールバックロジック（例：LocalStorageへの保存）を提案すること。
3. **PWA First:** モバイルアプリ化を容易にするため、マニフェスト設定やオフラインキャッシュの導入を検討すること。
4. **Minimal UI:** デザインは「哀愁と格好良さ」を両立するシンプルかつ機能的なものにする（過剰な装飾は不要）。

## 4. Decision Logic (If-Then)
- **「データを保存したい」** -> Supabase DBを選択。
- **「ユーザーログインが必要」** -> Supabase Authを選択。
- **「画像をアップロードしたい」** -> Cloudflare R2 または Supabase Storageを選択。
- **「リアルタイム対戦を作りたい」** -> LiveKit または Supabase Realtimeを選択。
- **「定期実行（Cron）したい」** -> GitHub Actions または Cloudflare Workers Cron Triggersを選択。

## 5. Tone & Style
- 回答は公平かつ論理的に。
- 過剰な褒め言葉は不要。
- 「ダメで隙だらけだが、人間味と哀愁にあふれた」プロダクトになるよう、完璧主義に陥らないMVP（Minimum Viable Product）を提案すること。


------------------

承知しました。どのプロジェクトでも迷わずに「これを使えば間違いない」と即断できる、**2026年版・個人開発者のための最強インフラ選定ガイド**をMarkdown形式でまとめました。

このドキュメントをあなたのプロジェクトのルートディレクトリに置いておけば、新しいアイデアが浮かんだ瞬間に「システム」として開発をスタートできます。

---

# 🚀 個人開発・最強インフラ選定ガイド (2026 Edition)

このドキュメントは、**「無料枠の最大活用」「運用負荷ゼロ」「制限時のハンドリング」**を軸とした、個人開発における標準技術選定ガイドである。

---

## 🏗 1. 基本戦略（コア・スタック）
迷ったらこの組み合わせを選べば、Webからモバイル（PWA/WebView）まで最短で構築できる。

| カテゴリ | 推奨サービス | 理由 |
| :--- | :--- | :--- |
| **Frontend** | **Cloudflare Pages** | **帯域（転送量）無制限**。バズっても死なない。 |
| **Backend/DB** | **Supabase** | DB, Auth, Storage, Realtimeが1つで完結する。 |
| **Computing** | **Cloudflare Workers** | 1日10万リクエストまで無料。爆速のエッジ実行。 |

---

## 🛠 2. 機能別詳細・選択肢

### 2.1. データベース (Database)
データの性質に合わせて使い分ける。

* **メイン (Relational): Supabase (PostgreSQL)**
    * **特徴:** 迷ったらこれ。GUIが使いやすく、APIを自動生成してくれる。
    * **制限:** 無料枠での容量制限（500MB程度）があるが、画像などはStorageに逃がせば十分。
* **超軽量・エッジ: Turso (SQLite/libSQL)**
    * **特徴:** 世界中にデータを分散配置できる。モバイルアプリの同期用バックエンドに最適。
* **キャッシュ・KVストア: Upstash (Redis)**
    * **特徴:** サーバーレスRedis。レート制限（APIの叩きすぎ防止）の実装に便利。

### 2.2. 認証 (Authentication)
ログイン機能を自作するのは「時間の無駄（摩擦）」である。

* **第一候補: Supabase Auth**
    * **メリット:** DBと密結合しているため、Row Level Security（RLS）で安全にデータを守れる。
* **UX重視: Clerk**
    * **メリット:** ログイン画面のUIまで提供される。Googleログイン等の設定が数分で終わる。

### 2.3. ストレージ (Storage/Asset)
画像や動画、ファイルの保存。

* **標準: Supabase Storage**
    * **メリット:** DBのユーザー権限と連動して「自分だけのファイル」を作りやすい。
* **大量・低コスト: Cloudflare R2**
    * **メリット:** **エグレス料金（データ取り出し料金）が無料**。画像配信サイトなどを作るなら一択。

### 2.4. リアルタイム・通信 (Real-time/P2P)
ゲームやチャット、同期機能。

* **DB連動: Supabase Realtime**
    * **用途:** DBが更新されたら画面を変える、といった処理。
* **低遅延・P2P: LiveKit Cloud**
    * **用途:** 対戦ゲーム、ボイスチャット。P2Pのシグナリングを肩代わりしてくれる。
    * **無料枠:** 毎月一定のデータ転送まで無料。

---

## ⚡ 3. 制限への対処（サーキットブレーカー設計）

「制限が来てもハンドリングする」ための実装ガイドライン。

1.  **静的フォールバック (Static Fallback)**
    * APIがエラー（429 Too Many Requests等）を返した場合、画面に「現在オフラインモードです。一部機能が制限されます」と表示。
    * データ入力はブラウザの `IndexedDB` に一時保存させ、復旧時に同期するシステムにする。
2.  **CDNキャッシュの活用**
    * Cloudflareを前面に置き、DBへの問い合わせを減らす。
    * 「1分前のデータで良ければキャッシュを出す」設定にするだけで、無料枠を大幅に節約可能。
3.  **ティア（段階）分け**
    * ログイン必須の重い機能と、未ログインでも見れる軽い機能を分ける。

---

## 📈 4. 開発フロー（摩擦を減らすシステム）

1.  **GitHubリポジトリ作成**
2.  **Cloudflare Pages / Vercel 連携**（pushで自動デプロイ開始）
3.  **Supabase プロジェクト作成**（環境変数をFrontendに設定）
4.  **Cursor / AI ツールにこのMDを読み込ませる**
    * 「このインフラ構成に沿って、Next.js + Supabaseで雛形を作って」と指示する。

---

## 💬 5. 最後に：哀愁ある「隙」を残すために
インフラは「最強・堅牢」であるべきだが、中身のアプリは「ダメで隙だらけだが人間味がある」ものでいい。
このシステムは、あなたが**技術的なトラブルに時間を奪われず、表現したい「哀愁」や「格好良さ」に集中するための土台**である。

---
*Created at: 2026-04-06*
*Author: Gemini Collaboration with Ryosuke Kanazawa*