# 残高スナップショットAPI

残高スナップショット（BalanceSnapshot）に関する操作を提供するAPIエンドポイント。

---

## エンドポイント一覧

### 1. 残高スナップショット一覧取得（管理者専用）

指定した政治団体の残高スナップショットを取得します。

**エンドポイント**
```
GET /api/balance-snapshots
```

**認証**: 必須（管理者のみ）

**クエリパラメータ**
```typescript
{
  politicalOrganizationId: string;     // 必須: 政治団体ID
  sortBy?: "date" | "balance";         // ソート基準（デフォルト: date）
  order?: "asc" | "desc";              // ソート順（デフォルト: desc）
}
```

**リクエスト例**
```
GET /api/balance-snapshots?politicalOrganizationId=1&sortBy=date&order=desc
```

**レスポンス**
```typescript
{
  balanceSnapshots: BalanceSnapshot[];
  politicalOrganization: PoliticalOrganization;
}
```

**レスポンス例**
```json
{
  "balanceSnapshots": [
    {
      "id": "1",
      "politicalOrganizationId": "1",
      "snapshotDate": "2024-12-31T00:00:00.000Z",
      "balance": 10000000,
      "createdAt": "2024-12-31T23:59:00.000Z",
      "updatedAt": "2024-12-31T23:59:00.000Z"
    },
    {
      "id": "2",
      "politicalOrganizationId": "1",
      "snapshotDate": "2023-12-31T00:00:00.000Z",
      "balance": 8500000,
      "createdAt": "2023-12-31T23:59:00.000Z",
      "updatedAt": "2023-12-31T23:59:00.000Z"
    }
  ],
  "politicalOrganization": {
    "id": "1",
    "displayName": "山田太郎",
    "slug": "yamada-taro",
    "orgName": "山田太郎後援会",
    "description": "XX県選出の衆議院議員",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-15T12:30:00.000Z"
  }
}
```

**ステータスコード**
- `200 OK`: 成功
- `400 Bad Request`: パラメータエラー
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `404 Not Found`: 指定された政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

---

### 2. 残高スナップショット詳細取得（管理者専用）

残高スナップショットの詳細情報を取得します。

**エンドポイント**
```
GET /api/balance-snapshots/:id
```

**認証**: 必須（管理者のみ）

**パスパラメータ**
- `id` (string): 残高スナップショットID

**レスポンス**
```typescript
{
  balanceSnapshot: BalanceSnapshot;
}
```

**レスポンス例**
```json
{
  "balanceSnapshot": {
    "id": "1",
    "politicalOrganizationId": "1",
    "snapshotDate": "2024-12-31T00:00:00.000Z",
    "balance": 10000000,
    "createdAt": "2024-12-31T23:59:00.000Z",
    "updatedAt": "2024-12-31T23:59:00.000Z"
  }
}
```

**ステータスコード**
- `200 OK`: 成功
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `404 Not Found`: 指定されたIDの残高スナップショットが存在しない
- `500 Internal Server Error`: サーバーエラー

---

### 3. 残高スナップショット作成（管理者専用）

新しい残高スナップショットを作成します。

**エンドポイント**
```
POST /api/balance-snapshots
```

**認証**: 必須（管理者のみ）

**リクエストボディ**
```typescript
{
  politicalOrganizationId: string;     // 必須: 政治団体ID
  snapshotDate: string;                // 必須: スナップショット日付（YYYY-MM-DD）
  balance: number;                     // 必須: 残高
}
```

**リクエスト例**
```json
{
  "politicalOrganizationId": "1",
  "snapshotDate": "2024-12-31",
  "balance": 10000000
}
```

**レスポンス**
```typescript
{
  balanceSnapshot: BalanceSnapshot;
}
```

**レスポンス例**
```json
{
  "balanceSnapshot": {
    "id": "3",
    "politicalOrganizationId": "1",
    "snapshotDate": "2024-12-31T00:00:00.000Z",
    "balance": 10000000,
    "createdAt": "2024-12-31T10:00:00.000Z",
    "updatedAt": "2024-12-31T10:00:00.000Z"
  }
}
```

**ステータスコード**
- `201 Created`: 作成成功
- `400 Bad Request`: バリデーションエラー（政治団体IDが未指定、残高が負の値など）
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `404 Not Found`: 指定された政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

**バリデーションルール**
- `politicalOrganizationId`: 必須、存在する政治団体のID
- `snapshotDate`: 必須、有効な日付形式（YYYY-MM-DD）
- `balance`: 必須、数値型（Decimal(15,2)）

**注意事項**
- 同一政治団体・同一日付で複数の残高スナップショットを作成可能
- 最新のupdatedAtを持つスナップショットが「最新残高」として扱われる

---

### 4. 残高スナップショット更新（管理者専用）

既存の残高スナップショットを更新します。

**エンドポイント**
```
PUT /api/balance-snapshots/:id
PATCH /api/balance-snapshots/:id
```

**認証**: 必須（管理者のみ）

**パスパラメータ**
- `id` (string): 残高スナップショットID

**リクエストボディ**
```typescript
{
  snapshotDate?: string;               // スナップショット日付（YYYY-MM-DD）
  balance?: number;                    // 残高
}
```

**リクエスト例**
```json
{
  "balance": 10500000
}
```

**レスポンス**
```typescript
{
  balanceSnapshot: BalanceSnapshot;
}
```

**レスポンス例**
```json
{
  "balanceSnapshot": {
    "id": "3",
    "politicalOrganizationId": "1",
    "snapshotDate": "2024-12-31T00:00:00.000Z",
    "balance": 10500000,
    "createdAt": "2024-12-31T10:00:00.000Z",
    "updatedAt": "2024-12-31T15:20:00.000Z"
  }
}
```

**ステータスコード**
- `200 OK`: 更新成功
- `400 Bad Request`: バリデーションエラー
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `404 Not Found`: 指定されたIDの残高スナップショットが存在しない
- `500 Internal Server Error`: サーバーエラー

---

### 5. 残高スナップショット削除（管理者専用）

残高スナップショットを削除します。

**エンドポイント**
```
DELETE /api/balance-snapshots/:id
```

**認証**: 必須（管理者のみ）

**パスパラメータ**
- `id` (string): 残高スナップショットID

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
  "message": "Balance snapshot deleted successfully"
}
```

**ステータスコード**
- `200 OK`: 削除成功
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `404 Not Found`: 指定されたIDの残高スナップショットが存在しない
- `500 Internal Server Error`: サーバーエラー

**注意事項**
- この操作は元に戻せません
- 削除後、貸借対照表の計算結果が変わる可能性があります
- キャッシュが無効化されます

---

### 6. 最新残高取得（公開API）

指定した政治団体の最新残高を取得します。

**エンドポイント**
```
GET /api/balance-snapshots/latest
```

**認証**: 不要

**クエリパラメータ**
```typescript
{
  slugs: string[];           // 必須: 政治団体のSlug（複数指定可）
}
```

**リクエスト例**
```
GET /api/balance-snapshots/latest?slugs=yamada-taro&slugs=tanaka-hanako
```

**レスポンス**
```typescript
{
  totalBalance: number;                // 合計残高
  balancesByOrganization: Array<{
    organizationId: string;
    organizationName: string;
    slug: string;
    balance: number;
    snapshotDate: string;
  }>;
}
```

**レスポンス例**
```json
{
  "totalBalance": 15000000,
  "balancesByOrganization": [
    {
      "organizationId": "1",
      "organizationName": "山田太郎後援会",
      "slug": "yamada-taro",
      "balance": 10000000,
      "snapshotDate": "2024-12-31T00:00:00.000Z"
    },
    {
      "organizationId": "2",
      "organizationName": "田中花子政治資金団体",
      "slug": "tanaka-hanako",
      "balance": 5000000,
      "snapshotDate": "2024-12-31T00:00:00.000Z"
    }
  ]
}
```

**ステータスコード**
- `200 OK`: 成功
- `400 Bad Request`: パラメータエラー
- `404 Not Found`: 指定されたSlugの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

**用途**
- トップページでの残高表示
- 貸借対照表の流動資産計算

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
      "balance": "Balance must be a positive number",
      "snapshotDate": "Invalid date format"
    }
  }
}
```

---

## 使用例（フロントエンド）

### 一覧取得（管理者）
```typescript
const response = await fetch('/api/balance-snapshots?politicalOrganizationId=1', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
const { balanceSnapshots, politicalOrganization } = await response.json();
```

### 作成（管理者）
```typescript
const response = await fetch('/api/balance-snapshots', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    politicalOrganizationId: '1',
    snapshotDate: '2024-12-31',
    balance: 10000000
  })
});
const { balanceSnapshot } = await response.json();
```

### 更新（管理者）
```typescript
const response = await fetch('/api/balance-snapshots/3', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    balance: 10500000
  })
});
const { balanceSnapshot } = await response.json();
```

### 削除（管理者）
```typescript
const response = await fetch('/api/balance-snapshots/3', {
  method: 'DELETE',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
const { success, message } = await response.json();
```

### 最新残高取得（公開）
```typescript
const params = new URLSearchParams();
params.append('slugs', 'yamada-taro');
params.append('slugs', 'tanaka-hanako');

const response = await fetch(`/api/balance-snapshots/latest?${params}`);
const { totalBalance, balancesByOrganization } = await response.json();
```

---

## データベース操作

現在の実装では、以下のUsecaseとRepositoryを使用しています：

**Usecase**
- `CreateBalanceSnapshotUsecase`: 作成
- `DeleteBalanceSnapshotUsecase`: 削除
- （一覧取得・更新Usecaseは今後実装）

**Repository**
- `PrismaBalanceSnapshotRepository`
  - `findByPoliticalOrganizationId(orgId)`: 政治団体IDで検索
  - `create(data)`: 作成
  - `update(id, data)`: 更新
  - `delete(id)`: 削除
  - `getTotalLatestBalanceByOrgIds(orgIds[])`: 最新残高合計取得
  - `getTotalLatestBalancesByYear(orgIds[], financialYear)`: 会計年度別最新残高取得

---

## キャッシュ戦略

### 一覧取得
- **キャッシュ時間**: 60秒
- **キャッシュキー**: `["balance-snapshots-data", politicalOrganizationId]`

### 最新残高取得
- **キャッシュ時間**: 300秒（5分）
- **キャッシュキー**: `["latest-balance", slugs]`

### 作成/更新/削除
- **キャッシュ無効化**: revalidateTag

現在の実装:
```typescript
export const loadBalanceSnapshotsData = unstable_cache(
  async (politicalOrganizationId: string) => {
    // ... データ取得処理
  },
  ["balance-snapshots-data"],
  { revalidate: 60 },
);
```

API化後は、Redisなどを使った本格的なキャッシュ戦略を検討します。

---

## ビジネスロジック

### 最新残高の定義
- 同一政治団体・同一日付で複数のスナップショットが存在する場合、最新の `updatedAt` を持つものを使用
- データベースインデックス: `(politicalOrganizationId, snapshotDate DESC, updatedAt DESC)`

### 貸借対照表との連携
- 貸借対照表の「流動資産」は、最新残高スナップショットの合計
- 会計年度ごとの残高比較に使用

### データ入力タイミング
- 会計年度開始時（例: 2024-01-01）
- 会計年度終了時（例: 2024-12-31）
- 月次締め時（任意）

---

## 注意事項

### データ整合性
- 残高スナップショットは手動入力が前提
- トランザクションデータとの自動同期は行われない
- 定期的な残高照合が必要

### 複数組織対応
- 複数の政治団体の残高を合算して表示可能
- slug配列で複数指定可能

### 削除時の影響
- 残高スナップショットを削除すると、貸借対照表の計算結果が変わる
- 削除前に影響範囲を確認することを推奨
