# 結婚式ゲスト向け写真共有Webアプリ 要件定義

---

## 前提・ポリシー
- Driveフォルダは「リンクを知っている全員・編集可」だが、API経由の操作は常にサービスアカウントで実行。
- サービスアカウントを対象フォルダの編集者として追加し、操作履歴は当該アカウント名義で残る。
- アップロードは「フロント→API→Drive」に限定（ブラウザからDrive直接は不採用）。

---

## 1. プロジェクト概要
- 結婚式ゲスト（30〜40人）向けの写真共有Webアプリ。
- Webアプリ上で写真のアップロード、閲覧、コメント、いいねが可能。
- 裏では **Google Drive** をストレージとして利用。
- 最悪、アプリが利用できない場合は、Driveの共有リンクだけで写真閲覧・アップロードが可能。

---

## 2. システム構成
[React Frontend (Vercel)]
│ axios/fetch
▼
[Spring Boot API (Cloud Run)]
├─ Google OAuth2 認証（※いいね・コメント投稿時のみ必須）
├─ Google Drive API 操作
└─ DB (Supabase / Cloud SQL)
├─ photo
├─ user
└─ like


- **フロント**: React（Vite または Next.js）、Vercelホスティング
- **API**: Spring Boot、Cloud Runデプロイ
- **DB**: コメント・いいね管理用（Supabase推奨）
- **ストレージ**: Google Drive（ホストの共有フォルダ）
- **認証**: Google OAuth2

---

## 3. 主な機能要件

| 機能 | 詳細 |
|------|------|
| 写真アップロード | Reactアプリからのみ対応、iPhone撮影写真を想定 (`.jpg`, `.jpeg`, `.png`, `.heic`)、最大10MB。原本は無変換保存、EXIF回転のみサーバーで正規化。10MB超は413応答または長辺~4000px縮小推奨。 |
| 写真閲覧 | Drive APIのサムネイルURL（thumbnailLink）で表示、原本閲覧はwebViewLinkへディープリンク。 |
| 写真削除 | API経由削除のみ許可、DriveとDB両方を更新。監査ログ保存。 |
| コメント | Driveコメント機能（閲覧は誰でも可、投稿はGoogleログイン必須）。 |
| いいね | DB保存（photoテーブルと紐付け）、Googleログイン必須。 |
| 双方向同期 | Drive ↔ DB（changes.listで差分同期、障害時はフル再取込）。 |
| ゲストモード | Googleアカウント未保持者は簡易アップロード可能（優先度低）。 |

---

## 認証・認可
- アップロード／閲覧は認証不要。
- いいねはGoogleログイン必須（DBのuser_google_idと紐付け）。
- コメントは閲覧は誰でも可、投稿はGoogleログイン必須（Drive Comments API使用）。

---

## 4. 非機能要件

| 項目 | 内容 |
|------|------|
| 同時アクセス | 最大40人想定 |
| ファイルサイズ制限 | 写真最大10MB、動画はDrive直接アップロード |
| フォルダ構成 | 単一フォルダ固定 (`WeddingApp/photos`) |
| サムネイル | Drive APIの `thumbnailLink` 使用 |
| トークン管理 | GitHub Secrets経由でAPIトークン管理 |
| 障害対応 | アプリが使えない場合はDriveリンクをメール等で共有 |
| MIME/サイズ検証 | サーバー側で厳密検証 |
| EXIF正規化 | サーバーで自動回転 |
| レート制限 | 1IPあたり簡易レート制限 |
| 削除監査 | API経由削除のみ許可、監査ログ保存 |
| フォルダリンク漏洩リスク | 前提として明記 |
| デプロイ例 | フロントはVercel、APIはCloud Run、DBはSupabase推奨 |

---

## 5. DB設計（抽出情報・補足）

### photo

| カラム名 | 型 | 説明 |
|-----------|----|------|
| id | BIGINT (PK) | アプリ内一意キー |
| drive_file_id | VARCHAR(255) | Google DriveファイルID（Drive取得値）|
| file_name | VARCHAR(255) | ファイル名 |
| file_url | TEXT | 共有URL（Drive取得値）|
| uploader_google_id | VARCHAR(255) | アップロード者のGoogleアカウントID（NULL可、匿名アップロード許容、user.google_id外部キー）|
| uploader_name | VARCHAR(100) | 表示名 |
| uploaded_at | DATETIME | アップロード日時（インデックス: DESC）|
| thumbnail_url | TEXT | サムネイルURL（Drive取得値）|
| mime_type | VARCHAR(50) | MIMEタイプ（Drive取得値）|
| size_bytes | BIGINT | ファイルサイズ（Drive取得値）|
| deleted | BOOLEAN | 論理削除フラグ（Drive側削除との整合）|

### like

| カラム名 | 型 | 説明 |
|-----------|----|------|
| id | BIGINT (PK) | 一意キー |
| photo_id | BIGINT (FK → photo.id) | 対象写真（外部キー、ON DELETE CASCADE/RESTRICT、インデックス）|
| user_google_id | VARCHAR(255) | いいねしたユーザーのGoogle ID（インデックス）|
| created_at | DATETIME | いいね日時 |
- 一意制約: UNIQUE(photo_id, user_google_id)

### user（オプション）

| カラム名 | 型 | 説明 |
|-----------|----|------|
| google_id | VARCHAR(255) (PK) | GoogleアカウントID |
| display_name | VARCHAR(100) | 表示名 |
| email | VARCHAR(255) | メールアドレス（Drive権限チェック用） |
| avatar_url | TEXT | プロフィール画像URL |
| last_login_at | DATETIME | 最終ログイン日時 |

### drive_sync_token（メタテーブル）

| カラム名 | 型 | 説明 |
|-----------|----|------|
| key | VARCHAR(100) (PK) | メタ情報キー（例: startPageToken）|
| value | TEXT | 値 |
- changes.listのstartPageTokenを保持

---

## 6. Drive API利用方針
- 使用API: files.list／files.get／files.create／files.delete／changes.list
- fields最適化: id, name, mimeType, size, thumbnailLink, webViewLink, createdTime
- changes.listのstartPageTokenをDBに保持し、定期ジョブで差分同期。障害時のフル再取込パスを用意
- コメントはDrive Comments API管理（DB非保持）

---

## 7. API仕様（最小）
- GET /api/photos?cursor=&limit=
- POST /api/photos
- DELETE /api/photos/{id}
- POST /api/photos/{id}/likes
- DELETE /api/photos/{id}/likes
- GET /api/photos/{id}/comments
- POST /api/photos/{id}/comments

---

## 8. 環境変数
- GOOGLE_DRIVE_FOLDER_ID
- GOOGLE_APPLICATION_CREDENTIALS（サービスアカウント鍵の参照パス or シークレット注入）
- GOOGLE_CLIENT_ID／GOOGLE_CLIENT_SECRET
- SPRING_DATASOURCE_*
- 任意: OAUTH_ALLOWED_EMAIL_DOMAIN

---

## 9. 今後の検討項目
1. ER図化して関係性を可視化
2. Spring Boot側のエンティティ／リポジトリ設計
3. Cloud Run & Vercel環境変数設定（DriveフォルダID、OAuthトークン）
4. 双方向同期の実装方法（定期ジョブ or webhook）
