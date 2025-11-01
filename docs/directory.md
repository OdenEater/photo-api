# ディレクトリ構成・ファイル役割

```
wedding-photo-api/
├── README.md                # 要件・仕様まとめ
├── docs/
│   ├── ER.md                # ER図・DB設計
│   └── directory.md         # ディレクトリ構成・ファイル役割
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── org/example/
│   │   │       ├── controller/
│   │   │       │   ├── PhotoController.java      # 写真API（一覧・アップロード・削除）
│   │   │       │   ├── LikeController.java       # いいねAPI
│   │   │       │   ├── CommentController.java    # コメントAPI（Drive連携）
│   │   │       ├── service/
│   │   │       │   ├── PhotoService.java         # 写真操作ロジック
│   │   │       │   ├── LikeService.java          # いいね操作ロジック
│   │   │       │   ├── CommentService.java       # コメント操作ロジック
│   │   │       │   ├── DriveService.java         # Drive API連携
│   │   │       ├── repository/
│   │   │       │   ├── PhotoRepository.java      # 写真DBアクセス
│   │   │       │   ├── LikeRepository.java       # いいねDBアクセス
│   │   │       │   ├── UserRepository.java       # ユーザーDBアクセス
│   │   │       │   ├── DriveSyncTokenRepository.java # Drive同期トークン管理
│   │   │       ├── model/
│   │   │       │   ├── Photo.java                # 写真エンティティ
│   │   │       │   ├── Like.java                 # いいねエンティティ
│   │   │       │   ├── User.java                 # ユーザーエンティティ
│   │   │       │   ├── DriveSyncToken.java       # 同期トークンエンティティ
│   │   │       ├── config/
│   │   │       │   ├── SecurityConfig.java       # 認証・認可設定
│   │   │       │   ├── DriveConfig.java          # Drive API設定
│   │   │       ├── Main.java                     # Spring Boot起動
│   │   ├── resources/
│   │   │   ├── application.yml                   # 環境変数・DB設定
│   │   │   ├── schema.sql                        # DB初期スキーマ
│   ├── test/
│   │   └── java/
│   │       └── org/example/
│   │           ├── controller/                   # コントローラーテスト
│   │           ├── service/                      # サービステスト
│   │           ├── repository/                   # リポジトリテスト
├── frontend/                  # Reactフロントエンド（Vite/Next.js等）
│   ├── src/
│   │   ├── components/
│   │   │   ├── PhotoList.tsx         # 写真一覧表示
│   │   │   ├── PhotoUpload.tsx       # 写真アップロード
│   │   │   ├── PhotoDetail.tsx       # 写真詳細・コメント・いいね
│   │   │   ├── LikeButton.tsx        # いいねボタン
│   │   │   ├── CommentList.tsx       # コメント表示
│   │   │   ├── CommentForm.tsx       # コメント投稿
│   │   ├── pages/
│   │   │   ├── index.tsx             # トップページ
│   │   │   ├── photos.tsx            # 写真一覧ページ
│   │   │   ├── photo/[id].tsx        # 写真詳細ページ
│   │   ├── utils/
│   │   │   ├── api.ts                # API通信ラッパー
│   │   │   ├── auth.ts               # Google認証処理
│   │   │   ├── drive.ts              # Drive API補助
│   │   ├── App.tsx                   # ルートコンポーネント
│   │   ├── main.tsx                  # エントリポイント
│   ├── public/
│   │   ├── index.html                # HTMLテンプレート
│   ├── package.json                  # フロント依存管理
│   ├── vite.config.ts                # Vite設定（Next.jsなら next.config.js）
```

---

## 各ディレクトリ・ファイルの役割

### docs/
- ER.md: DB設計・ER図
- directory.md: ディレクトリ構成と役割まとめ

### src/main/java/org/example/
- controller/: REST APIエンドポイント（リクエスト受付・レスポンス返却）
- service/: 業務ロジック（Drive連携・認証・EXIF処理など）
- repository/: DBアクセス（CRUD・検索・同期トークン管理）
- model/: エンティティ定義（DBテーブル・APIレスポンス用）
- config/: 認証・Drive・DB設定（Spring Security, OAuth, Drive APIなど）
- Main.java: Spring Bootアプリケーション起動

### src/main/resources/
- application.yml: 環境変数・DB接続設定
- schema.sql: DB初期スキーマ

### src/test/java/org/example/
- controller/: コントローラーテスト
- service/: サービステスト
- repository/: リポジトリテスト

### frontend/
- components/: React UI部品（一覧・アップロード・詳細・いいね・コメント等）
- pages/: 画面単位のページ（トップ・一覧・詳細等）
- utils/: API通信・認証・Drive連携の補助関数
- App.tsx, main.tsx: ルート・エントリポイント
- public/: 静的ファイル（HTML等）
- package.json: フロント依存管理
- vite.config.ts: Vite設定（Next.jsなら next.config.js）

---

この構成をベースに、各機能ごとにファイルを分割・実装していくことが推奨です。

