+------------+ 1 N +-----------+
| user |------------------------| photo |
+------------+ +-----------+
| google_id PK| | id PK |
| display_name| | drive_file_id |
| email | | file_name |
| avatar_url | | file_url |
| last_login_at | | uploader_google_id FK -> user.google_id (NULL可, ユーザー削除時 SET NULL/RESTRICT)
+------------+ | uploader_name |
| uploaded_at |
| thumbnail_url |
| mime_type |
| size_bytes |
| deleted | (論理削除, Drive側削除と整合)
+-----------+
| インデックス: uploaded_at DESC, drive_file_id, uploader_google_id |
|
| 1
|
| N
+-----------+
| like |
+-----------+
| id PK |
| photo_id FK -> photo.id (ON DELETE CASCADE/RESTRICT, インデックス) |
| user_google_id | (インデックス)
| created_at |
| UNIQUE(photo_id, user_google_id) |
+-----------+

### drive_sync_token（メタテーブル）
| key PK | value |
- changes.list の startPageToken を保持

### 説明
- **user**: Googleログインユーザー情報をキャッシュ
- **photo**: アップロードされた写真情報（DriveファイルID保持, Drive取得値を保持: thumbnail_url, file_url, mime_type, size_bytes）
- **like**: 写真に対するいいね情報（photo_id, user_google_idで一意制約）
- **関係**
    - 1人のユーザーは複数の写真をアップロード可能（1:N）
    - 1枚の写真に複数のいいねが可能（1:N）
- コメントはDrive APIで管理するためDBには持たない設計（DB非保持を明示）
- deletedは論理削除。Drive側削除と整合を取る方針。
- 外部キー制約・インデックス・同期メタテーブル設計を明記。
