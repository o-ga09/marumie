# APIエンドポイント一覧

すべてのAPIエンドポイントの概要と認証要件を一覧表示します。

---

## ベースURL

### 開発環境
```
http://localhost:4000/api
```

### 本番環境
```
https://api.marumie.example.com/api
```

---

## エンドポイント一覧表

### 認証API (`/api/auth`)

| メソッド | エンドポイント | 説明 | 認証 | 権限 |
|---------|---------------|------|------|------|
| POST | `/auth/signup` | ユーザー登録 | 不要 | - |
| POST | `/auth/login` | ログイン | 不要 | - |
| POST | `/auth/logout` | ログアウト | 必須 | - |
| POST | `/auth/refresh` | トークンリフレッシュ | 不要 | - |
| GET | `/auth/me` | 現在のユーザー情報取得 | 必須 | - |
| POST | `/auth/change-password` | パスワード変更 | 必須 | - |
| POST | `/auth/reset-password` | パスワードリセット要求 | 不要 | - |

### 政治団体API (`/api/political-organizations`)

| メソッド | エンドポイント | 説明 | 認証 | 権限 |
|---------|---------------|------|------|------|
| GET | `/political-organizations` | 政治団体一覧取得 | 不要 | - |
| GET | `/political-organizations/:slug` | 政治団体詳細取得 | 不要 | - |
| POST | `/political-organizations` | 政治団体作成 | 必須 | 管理者 |
| PUT | `/political-organizations/:id` | 政治団体更新 | 必須 | 管理者 |
| PATCH | `/political-organizations/:id` | 政治団体部分更新 | 必須 | 管理者 |
| DELETE | `/political-organizations/:id` | 政治団体削除 | 必須 | 管理者 |

### トランザクションAPI (`/api/transactions`)

| メソッド | エンドポイント | 説明 | 認証 | 権限 |
|---------|---------------|------|------|------|
| GET | `/transactions` | トランザクション一覧取得 | 不要 | - |
| GET | `/transactions/:id` | トランザクション詳細取得 | 不要 | - |
| GET | `/transactions/export/csv` | トランザクションCSVエクスポート | 不要 | - |
| POST | `/transactions/preview` | CSVプレビュー | 必須 | 管理者 |
| POST | `/transactions/upload` | トランザクション一括登録 | 必須 | 管理者 |
| DELETE | `/transactions` | トランザクション全削除 | 必須 | 管理者 |

### データ集計API (`/api/analytics`)

| メソッド | エンドポイント | 説明 | 認証 | 権限 |
|---------|---------------|------|------|------|
| GET | `/analytics/top-page` | トップページデータ取得 | 不要 | - |
| GET | `/analytics/monthly` | 月次収支集計取得 | 不要 | - |
| GET | `/analytics/sankey` | サンキー図データ取得 | 不要 | - |
| GET | `/analytics/balance-sheet` | 貸借対照表データ取得 | 不要 | - |
| GET | `/analytics/daily-donations` | 日次寄付金推移取得 | 不要 | - |

### 残高スナップショットAPI (`/api/balance-snapshots`)

| メソッド | エンドポイント | 説明 | 認証 | 権限 |
|---------|---------------|------|------|------|
| GET | `/balance-snapshots` | 残高スナップショット一覧取得 | 必須 | 管理者 |
| GET | `/balance-snapshots/:id` | 残高スナップショット詳細取得 | 必須 | 管理者 |
| GET | `/balance-snapshots/latest` | 最新残高取得 | 不要 | - |
| POST | `/balance-snapshots` | 残高スナップショット作成 | 必須 | 管理者 |
| PUT | `/balance-snapshots/:id` | 残高スナップショット更新 | 必須 | 管理者 |
| PATCH | `/balance-snapshots/:id` | 残高スナップショット部分更新 | 必須 | 管理者 |
| DELETE | `/balance-snapshots/:id` | 残高スナップショット削除 | 必須 | 管理者 |

### ユーティリティAPI (`/api`)

| メソッド | エンドポイント | 説明 | 認証 | 権限 |
|---------|---------------|------|------|------|
| GET | `/health` | ヘルスチェック | 不要 | - |
| GET | `/version` | APIバージョン情報 | 不要 | - |
| POST | `/refresh` | キャッシュ無効化 | 必須 | 管理者 |

---

## 認証要件

### 公開エンドポイント（認証不要）
以下のエンドポイントは認証なしでアクセス可能：
- すべての `/auth` エンドポイント（ログイン、登録など）
- `/political-organizations` の GET
- `/transactions` の GET（一覧、詳細、CSVエクスポート）
- `/analytics` のすべての GET
- `/balance-snapshots/latest`
- `/health`, `/version`

### 管理者専用エンドポイント
以下のエンドポイントは管理者権限が必要：
- `/political-organizations` の POST, PUT, PATCH, DELETE
- `/transactions/preview`, `/transactions/upload`, `/transactions` の DELETE
- `/balance-snapshots` のすべて（`/latest` を除く）
- `/refresh`

---

## HTTPステータスコード

| コード | 説明 | 用途 |
|--------|------|------|
| 200 OK | 成功 | GET, PUT, PATCH, DELETE の成功レスポンス |
| 201 Created | 作成成功 | POST の成功レスポンス |
| 400 Bad Request | リクエストエラー | バリデーションエラー、パラメータ不足 |
| 401 Unauthorized | 認証エラー | トークン未提供、トークン無効 |
| 403 Forbidden | 権限不足 | 管理者以外がアクセス |
| 404 Not Found | リソース未発見 | 存在しないID、Slugを指定 |
| 409 Conflict | 競合 | ユニーク制約違反（メールアドレス重複など） |
| 429 Too Many Requests | レート制限超過 | API呼び出し回数制限 |
| 500 Internal Server Error | サーバーエラー | サーバー内部エラー |
| 503 Service Unavailable | サービス利用不可 | メンテナンス中 |

---

## レスポンス形式

### 成功レスポンス
```json
{
  "data": { /* レスポンスデータ */ },
  "meta": {
    "timestamp": "2024-08-20T10:30:00.000Z",
    "requestId": "req-abc123"
  }
}
```

### エラーレスポンス
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": { /* 詳細情報（任意） */ }
  },
  "meta": {
    "timestamp": "2024-08-20T10:30:00.000Z",
    "requestId": "req-abc123"
  }
}
```

---

## ページネーション

一覧取得エンドポイントでは以下のクエリパラメータをサポート：

```typescript
{
  page?: number;           // ページ番号（デフォルト: 1）
  perPage?: number;        // 1ページあたりの件数（デフォルト: 50、最大: 100）
}
```

レスポンスには以下のメタ情報を含む：
```json
{
  "data": [ /* データ */ ],
  "pagination": {
    "total": 1523,
    "page": 1,
    "perPage": 50,
    "totalPages": 31,
    "hasNext": true,
    "hasPrev": false
  }
}
```

---

## ソート

一覧取得エンドポイントでは以下のクエリパラメータをサポート：

```typescript
{
  sortBy?: string;         // ソート基準（例: "date", "amount"）
  order?: "asc" | "desc";  // ソート順（デフォルト: desc）
}
```

---

## フィルタリング

トランザクション一覧では以下のフィルタリングをサポート：

```typescript
{
  slugs: string[];                     // 政治団体のSlug（複数指定可）
  financialYear: number;               // 会計年度
  transactionType?: "income" | "expense"; // 取引種別
  dateFrom?: string;                   // 開始日（YYYY-MM-DD）
  dateTo?: string;                     // 終了日（YYYY-MM-DD）
  categories?: string[];               // カテゴリキー（複数指定可）
}
```

---

## レート制限

### 一般エンドポイント
- 1分あたり60リクエスト
- 1時間あたり1000リクエスト

### 認証エンドポイント
- ログイン: 15分あたり5リクエスト
- その他: 15分あたり20リクエスト

### 管理者専用エンドポイント
- 1分あたり100リクエスト
- 1時間あたり2000リクエスト

レート制限超過時のレスポンス：
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "details": {
      "limit": 60,
      "remaining": 0,
      "resetAt": "2024-08-20T10:31:00.000Z"
    }
  }
}
```

---

## CORS設定

### 許可されたオリジン
- 開発環境: `http://localhost:3000`, `http://localhost:3001`
- 本番環境: `https://marumie.example.com`, `https://admin.marumie.example.com`

### 許可されたメソッド
- GET, POST, PUT, PATCH, DELETE, OPTIONS

### 許可されたヘッダー
- Content-Type, Authorization, X-Requested-With

---

## バージョニング

APIバージョンはURLパスに含める：

```
/api/v1/political-organizations
/api/v2/political-organizations
```

現在のバージョン: **v1**

バージョン情報取得：
```
GET /api/version

{
  "version": "v1",
  "buildDate": "2024-08-20",
  "commit": "abc123"
}
```

---

## Webhook（将来実装予定）

トランザクション登録時やデータ更新時にWebhookを送信する機能を検討中。

```
POST /api/webhooks
{
  "url": "https://your-service.com/webhook",
  "events": ["transaction.created", "transaction.updated"],
  "secret": "your-webhook-secret"
}
```

---

## GraphQL（将来実装予定）

RESTに加えてGraphQL APIの提供を検討中。

```
POST /graphql
```

---

## 関連ドキュメント

- [概要](./00_overview.md)
- [データモデル](./01_data_models.md)
- [政治団体API](./02_political_organizations.md)
- [トランザクションAPI](./03_transactions.md)
- [データ集計API](./04_analytics.md)
- [残高スナップショットAPI](./05_balance_snapshots.md)
- [認証API](./06_authentication.md)
