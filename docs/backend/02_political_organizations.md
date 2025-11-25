# 政治団体API

政治団体（PoliticalOrganization）に関する操作を提供するAPIエンドポイント。

---

## エンドポイント一覧

### 1. 政治団体一覧取得

すべての政治団体を取得します（公開API）。

**エンドポイント**
```
GET /api/political-organizations
```

**認証**: 不要

**クエリパラメータ**
なし

**レスポンス**
```typescript
{
  organizations: PoliticalOrganization[];
}
```

**レスポンス例**
```json
{
  "organizations": [
    {
      "id": "1",
      "displayName": "山田太郎",
      "orgName": "山田太郎後援会",
      "slug": "yamada-taro",
      "description": "XX県選出の衆議院議員",
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-15T12:30:00.000Z"
    }
  ]
}
```

**ステータスコード**
- `200 OK`: 成功
- `500 Internal Server Error`: サーバーエラー

---

### 2. 政治団体詳細取得（Slug指定）

SlugまたはIDで政治団体を取得します（公開API）。

**エンドポイント**
```
GET /api/political-organizations/:slug
```

**認証**: 不要

**パスパラメータ**
- `slug` (string): 政治団体のSlug（例: `yamada-taro`）

**レスポンス**
```typescript
{
  organization: PoliticalOrganization;
}
```

**レスポンス例**
```json
{
  "organization": {
    "id": "1",
    "displayName": "山田太郎",
    "orgName": "山田太郎後援会",
    "slug": "yamada-taro",
    "description": "XX県選出の衆議院議員",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-15T12:30:00.000Z"
  }
}
```

**ステータスコード**
- `200 OK`: 成功
- `404 Not Found`: 指定されたSlugの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

---

### 3. 政治団体作成（管理者専用）

新しい政治団体を作成します。

**エンドポイント**
```
POST /api/political-organizations
```

**認証**: 必須（管理者のみ）

**リクエストボディ**
```typescript
{
  displayName: string;        // 必須
  slug: string;               // 必須（URLフレンドリーな文字列）
  orgName?: string;           // 任意
  description?: string;       // 任意
}
```

**リクエスト例**
```json
{
  "displayName": "田中花子",
  "slug": "tanaka-hanako",
  "orgName": "田中花子政治資金団体",
  "description": "YY県選出の参議院議員"
}
```

**レスポンス**
```typescript
{
  organization: PoliticalOrganization;
}
```

**レスポンス例**
```json
{
  "organization": {
    "id": "2",
    "displayName": "田中花子",
    "orgName": "田中花子政治資金団体",
    "slug": "tanaka-hanako",
    "description": "YY県選出の参議院議員",
    "createdAt": "2024-02-01T10:00:00.000Z",
    "updatedAt": "2024-02-01T10:00:00.000Z"
  }
}
```

**ステータスコード**
- `201 Created`: 作成成功
- `400 Bad Request`: バリデーションエラー（displayNameやslugが空、slugが重複など）
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `500 Internal Server Error`: サーバーエラー

**バリデーションルール**
- `displayName`: 必須、1文字以上
- `slug`: 必須、英数字とハイフンのみ、ユニーク
- `orgName`: 任意、255文字以内
- `description`: 任意

---

### 4. 政治団体更新（管理者専用）

既存の政治団体情報を更新します。

**エンドポイント**
```
PUT /api/political-organizations/:id
PATCH /api/political-organizations/:id
```

**認証**: 必須（管理者のみ）

**パスパラメータ**
- `id` (string): 政治団体のID

**リクエストボディ**
```typescript
{
  displayName?: string;
  orgName?: string | null;
  description?: string | null;
  slug?: string;
}
```

**リクエスト例**
```json
{
  "displayName": "田中花子",
  "description": "YY県選出の参議院議員（2期目）"
}
```

**レスポンス**
```typescript
{
  organization: PoliticalOrganization;
}
```

**レスポンス例**
```json
{
  "organization": {
    "id": "2",
    "displayName": "田中花子",
    "orgName": "田中花子政治資金団体",
    "slug": "tanaka-hanako",
    "description": "YY県選出の参議院議員（2期目）",
    "createdAt": "2024-02-01T10:00:00.000Z",
    "updatedAt": "2024-02-15T14:20:00.000Z"
  }
}
```

**ステータスコード**
- `200 OK`: 更新成功
- `400 Bad Request`: バリデーションエラー
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `404 Not Found`: 指定されたIDの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

---

### 5. 政治団体削除（管理者専用）

政治団体を削除します。関連するトランザクションと残高スナップショットも削除されます（CASCADE）。

**エンドポイント**
```
DELETE /api/political-organizations/:id
```

**認証**: 必須（管理者のみ）

**パスパラメータ**
- `id` (string): 政治団体のID

**レスポンス**
```typescript
{
  success: boolean;
  message: string;
}
```

**レスポンス例**
```json
{
  "success": true,
  "message": "Political organization deleted successfully"
}
```

**ステータスコード**
- `200 OK`: 削除成功
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `404 Not Found`: 指定されたIDの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

**注意事項**
- この操作は元に戻せません
- 関連する全トランザクションと残高スナップショットが削除されます

---

## エラーレスポンス

すべてのエンドポイントで共通のエラーレスポンス形式を使用します。

```typescript
{
  error: {
    code: string;           // エラーコード
    message: string;        // エラーメッセージ
    details?: any;          // 詳細情報（任意）
  }
}
```

**エラーレスポンス例**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {
      "displayName": "Display name is required",
      "slug": "Slug must contain only alphanumeric characters and hyphens"
    }
  }
}
```

---

## 使用例（フロントエンド）

### 一覧取得
```typescript
const response = await fetch('/api/political-organizations');
const { organizations } = await response.json();
```

### 詳細取得
```typescript
const response = await fetch('/api/political-organizations/yamada-taro');
const { organization } = await response.json();
```

### 作成（管理者）
```typescript
const response = await fetch('/api/political-organizations', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    displayName: '田中花子',
    slug: 'tanaka-hanako',
    orgName: '田中花子政治資金団体',
    description: 'YY県選出の参議院議員'
  })
});
const { organization } = await response.json();
```

### 更新（管理者）
```typescript
const response = await fetch('/api/political-organizations/2', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    description: 'YY県選出の参議院議員（2期目）'
  })
});
const { organization } = await response.json();
```

### 削除（管理者）
```typescript
const response = await fetch('/api/political-organizations/2', {
  method: 'DELETE',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
const { success, message } = await response.json();
```

---

## データベース操作

現在の実装では、以下のUsecaseとRepositoryを使用しています：

**Usecase**
- `CreatePoliticalOrganizationUsecase`
- `DeletePoliticalOrganizationUsecase`
- （更新Usecaseは今後実装）

**Repository**
- `PrismaPoliticalOrganizationRepository`
  - `findAll()`: 全件取得
  - `findBySlugs(slugs: string[])`: Slugで検索
  - `findById(id: string)`: IDで検索
  - `create(...)`: 作成
  - `update(...)`: 更新
  - `delete(id: string)`: 削除

---

## キャッシュ戦略

- **一覧取得**: 60秒キャッシュ（頻繁に変更されないため）
- **詳細取得**: 300秒キャッシュ（slug単位）
- **作成/更新/削除**: キャッシュ無効化（revalidateTag）

現在の実装:
```typescript
export const loadPoliticalOrganizationsData = unstable_cache(
  async (): Promise<PoliticalOrganization[]> => {
    // ...
  },
  ["political-organizations-data"],
  { revalidate: 60 },
);
```

API化後は、Redisなどを使った本格的なキャッシュ戦略を検討します。
