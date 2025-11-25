# トランザクションAPI

トランザクション（Transaction）に関する操作を提供するAPIエンドポイント。

---

## エンドポイント一覧

### 1. トランザクション一覧取得

指定した政治団体のトランザクションを取得します（公開API）。

**エンドポイント**
```
GET /api/transactions
```

**認証**: 不要

**クエリパラメータ**
```typescript
{
  slugs: string[];                     // 必須: 政治団体のSlug（複数指定可）
  financialYear: number;               // 必須: 会計年度
  page?: number;                       // ページ番号（デフォルト: 1）
  perPage?: number;                    // 1ページあたりの件数（デフォルト: 50、最大: 100）
  transactionType?: "income" | "expense"; // 取引種別フィルタ
  dateFrom?: string;                   // 開始日（YYYY-MM-DD）
  dateTo?: string;                     // 終了日（YYYY-MM-DD）
  sortBy?: "date" | "amount";          // ソート基準（デフォルト: date）
  order?: "asc" | "desc";              // ソート順（デフォルト: desc）
  categories?: string[];               // カテゴリキーでフィルタ
}
```

**リクエスト例**
```
GET /api/transactions?slugs=yamada-taro&financialYear=2024&page=1&perPage=50&transactionType=expense&sortBy=date&order=desc
```

**レスポンス**
```typescript
{
  transactions: DisplayTransaction[];
  total: number;
  page: number;
  perPage: number;
  totalPages: number;
  politicalOrganizations: PoliticalOrganization[];
  lastUpdatedAt: string | null;
}
```

**レスポンス例**
```json
{
  "transactions": [
    {
      "id": "12345",
      "date": "2024-08-15T00:00:00.000Z",
      "yearmonth": "2024.08",
      "transactionType": "expense",
      "category": "人件費",
      "subcategory": "給料手当",
      "account": "人件費",
      "label": "事務所スタッフ給与",
      "shortLabel": "スタッフ給与",
      "friendly_category": "人件費",
      "absAmount": 300000,
      "amount": -300000
    }
  ],
  "total": 1523,
  "page": 1,
  "perPage": 50,
  "totalPages": 31,
  "politicalOrganizations": [
    {
      "id": "1",
      "displayName": "山田太郎",
      "slug": "yamada-taro",
      "orgName": "山田太郎後援会",
      "description": "XX県選出の衆議院議員",
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-15T12:30:00.000Z"
    }
  ],
  "lastUpdatedAt": "2024-08-20T10:30:00.000Z"
}
```

**ステータスコード**
- `200 OK`: 成功
- `400 Bad Request`: パラメータエラー（slugsまたはfinancialYearが未指定など）
- `404 Not Found`: 指定されたSlugの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

---

### 2. トランザクション詳細取得

トランザクションの詳細情報を取得します（公開API）。

**エンドポイント**
```
GET /api/transactions/:id
```

**認証**: 不要

**パスパラメータ**
- `id` (string): トランザクションID

**レスポンス**
```typescript
{
  transaction: DisplayTransaction;
}
```

**レスポンス例**
```json
{
  "transaction": {
    "id": "12345",
    "date": "2024-08-15T00:00:00.000Z",
    "yearmonth": "2024.08",
    "transactionType": "expense",
    "category": "人件費",
    "subcategory": "給料手当",
    "account": "人件費",
    "label": "事務所スタッフ給与",
    "shortLabel": "スタッフ給与",
    "friendly_category": "人件費",
    "absAmount": 300000,
    "amount": -300000
  }
}
```

**ステータスコード**
- `200 OK`: 成功
- `404 Not Found`: 指定されたIDのトランザクションが存在しない
- `500 Internal Server Error`: サーバーエラー

---

### 3. トランザクションCSVエクスポート

指定した政治団体のトランザクションをCSV形式でエクスポートします（公開API）。

**エンドポイント**
```
GET /api/transactions/export/csv
```

**認証**: 不要

**クエリパラメータ**
```typescript
{
  slug: string;              // 必須: 政治団体のSlug
}
```

**リクエスト例**
```
GET /api/transactions/export/csv?slug=yamada-taro
```

**レスポンス**
- Content-Type: `text/csv; charset=utf-8`
- Content-Disposition: `attachment; filename="transactions_{slug}_{timestamp}.csv"`

**CSV形式**
```csv
日付,年月,種別,カテゴリ,サブカテゴリ,アカウント,ラベル,金額
2024-08-15,2024.08,支出,人件費,給料手当,人件費,事務所スタッフ給与,-300000
2024-08-10,2024.08,収入,寄付金,個人寄付,寄付金,個人寄付A氏,100000
```

**ステータスコード**
- `200 OK`: 成功
- `400 Bad Request`: パラメータエラー
- `404 Not Found`: 指定されたSlugの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

---

### 4. トランザクションCSVプレビュー（管理者専用）

CSVファイルをアップロードし、取込前にプレビューします。

**エンドポイント**
```
POST /api/transactions/preview
```

**認証**: 必須（管理者のみ）

**リクエストボディ**
- Content-Type: `multipart/form-data`

```typescript
{
  file: File;                          // CSVファイル
  politicalOrganizationId: string;     // 政治団体ID
}
```

**レスポンス**
```typescript
{
  transactions: PreviewTransaction[];
  summary: {
    total: number;                     // 総件数
    insert: number;                    // 新規登録件数
    update: number;                    // 更新件数
    skip: number;                      // スキップ件数（重複）
    invalid: number;                   // 無効件数（エラー）
  };
  statistics: {
    totalAmount: number;               // 合計金額
    incomeAmount: number;              // 収入合計
    expenseAmount: number;             // 支出合計
    transactionCount: number;          // 有効トランザクション数
  };
}
```

**レスポンス例**
```json
{
  "transactions": [
    {
      "transaction_no": "TX-2024-001",
      "transaction_date": "2024-08-15T00:00:00.000Z",
      "transaction_type": "expense",
      "debit_account": "人件費",
      "debit_sub_account": "給料手当",
      "debit_amount": 300000,
      "credit_account": "現金預金",
      "credit_sub_account": "",
      "credit_amount": 300000,
      "description": "事務所スタッフ給与",
      "label": "スタッフ給与",
      "friendly_category": "人件費",
      "category_key": "expense_personnel",
      "hash": "abc123...",
      "status": "insert",
      "validationErrors": [],
      "isDuplicate": false
    }
  ],
  "summary": {
    "total": 100,
    "insert": 85,
    "update": 5,
    "skip": 8,
    "invalid": 2
  },
  "statistics": {
    "totalAmount": 15000000,
    "incomeAmount": 8000000,
    "expenseAmount": 7000000,
    "transactionCount": 98
  }
}
```

**ステータスコード**
- `200 OK`: 成功
- `400 Bad Request`: ファイルが未指定、またはCSV形式が不正
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `500 Internal Server Error`: サーバーエラー

**CSV形式要件**
- 文字エンコーディング: Shift-JIS または UTF-8（BOM付き）
- ヘッダー行必須
- マネーフォワードクラウド会計のエクスポート形式に対応

---

### 5. トランザクション一括登録（管理者専用）

プレビュー済みのトランザクションを一括登録します。

**エンドポイント**
```
POST /api/transactions/upload
```

**認証**: 必須（管理者のみ）

**リクエストボディ**
```typescript
{
  validTransactions: PreviewTransaction[]; // status が "insert" or "update" のもの
  politicalOrganizationId: string;
}
```

**リクエスト例**
```json
{
  "validTransactions": [
    {
      "transaction_no": "TX-2024-001",
      "transaction_date": "2024-08-15T00:00:00.000Z",
      "transaction_type": "expense",
      "debit_account": "人件費",
      "debit_sub_account": "給料手当",
      "debit_amount": 300000,
      "credit_account": "現金預金",
      "credit_sub_account": "",
      "credit_amount": 300000,
      "description": "事務所スタッフ給与",
      "label": "スタッフ給与",
      "friendly_category": "人件費",
      "category_key": "expense_personnel",
      "hash": "abc123...",
      "status": "insert",
      "validationErrors": [],
      "isDuplicate": false
    }
  ],
  "politicalOrganizationId": "1"
}
```

**レスポンス**
```typescript
{
  ok: boolean;
  processedCount: number;
  savedCount: number;
  skippedCount: number;
  message: string;
  errors?: string[];
}
```

**レスポンス例**
```json
{
  "ok": true,
  "processedCount": 100,
  "savedCount": 95,
  "skippedCount": 5,
  "message": "100件を処理し、95件を保存、5件をスキップしました",
  "errors": []
}
```

**ステータスコード**
- `200 OK`: 成功（一部エラーがあっても200を返す）
- `400 Bad Request`: パラメータエラー
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `500 Internal Server Error`: サーバーエラー

**処理フロー**
1. `status` が `"insert"` のものは新規作成（bulkInsert）
2. `status` が `"update"` のものは更新（bulkUpdate）
3. `status` が `"skip"` のものは無視
4. 保存成功後、webappのキャッシュを無効化（`/api/refresh` を呼び出し）

---

### 6. トランザクション全削除（管理者専用）

指定した政治団体のトランザクションをすべて削除します。

**エンドポイント**
```
DELETE /api/transactions
```

**認証**: 必須（管理者のみ）

**クエリパラメータ**
```typescript
{
  politicalOrganizationId: string;     // 必須: 政治団体ID
}
```

**リクエスト例**
```
DELETE /api/transactions?politicalOrganizationId=1
```

**レスポンス**
```typescript
{
  success: boolean;
  deletedCount: number;
  message: string;
}
```

**レスポンス例**
```json
{
  "success": true,
  "deletedCount": 1523,
  "message": "All transactions deleted successfully"
}
```

**ステータスコード**
- `200 OK`: 削除成功
- `400 Bad Request`: パラメータエラー
- `401 Unauthorized`: 認証エラー
- `403 Forbidden`: 権限不足（管理者以外）
- `404 Not Found`: 指定された政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

**注意事項**
- この操作は元に戻せません
- 削除後、webappのキャッシュが無効化されます

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
    "message": "Invalid CSV format",
    "details": {
      "line": 15,
      "field": "debit_amount",
      "reason": "Amount must be a positive number"
    }
  }
}
```

---

## 使用例（フロントエンド）

### 一覧取得
```typescript
const params = new URLSearchParams({
  slugs: 'yamada-taro',
  financialYear: '2024',
  page: '1',
  perPage: '50',
  transactionType: 'expense',
  sortBy: 'date',
  order: 'desc'
});
const response = await fetch(`/api/transactions?${params}`);
const data = await response.json();
```

### CSVエクスポート
```typescript
const response = await fetch('/api/transactions/export/csv?slug=yamada-taro');
const blob = await response.blob();
const url = window.URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url;
a.download = 'transactions.csv';
a.click();
```

### CSVプレビュー（管理者）
```typescript
const formData = new FormData();
formData.append('file', file);
formData.append('politicalOrganizationId', '1');

const response = await fetch('/api/transactions/preview', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`
  },
  body: formData
});
const preview = await response.json();
```

### 一括登録（管理者）
```typescript
const response = await fetch('/api/transactions/upload', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    validTransactions: previewTransactions.filter(t => 
      t.status === 'insert' || t.status === 'update'
    ),
    politicalOrganizationId: '1'
  })
});
const result = await response.json();
```

---

## データベース操作

現在の実装では、以下のUsecaseとRepositoryを使用しています：

**Usecase**
- `GetTransactionsBySlugUsecase`: 一覧取得
- `GetAllTransactionsBySlugUsecase`: 全件取得（CSVエクスポート用）
- `PreviewMfCsvUsecase`: CSVプレビュー
- `SavePreviewTransactionsUsecase`: 一括登録
- `DeleteAllTransactionsUsecase`: 全削除

**Repository**
- `PrismaTransactionRepository`
  - `findById(id: string)`: 詳細取得
  - `findWithPagination(filters, pagination)`: ページネーション付き一覧
  - `findAll(filters)`: 全件取得
  - `createMany(transactions[])`: 一括作成
  - `updateMany(transactions[])`: 一括更新
  - `deleteAllByPoliticalOrganizationId(orgId)`: 全削除
  - `findByTransactionNos(nos[], orgIds[])`: 取引番号で検索（重複チェック）

---

## キャッシュ戦略

- **一覧取得**: 3600秒キャッシュ（slugs + financialYear + filters単位）
- **詳細取得**: キャッシュなし（頻繁にアクセスされないため）
- **CSVエクスポート**: キャッシュなし（動的生成）
- **CSVプレビュー**: キャッシュなし（リクエストごとに変わる）
- **一括登録/削除**: キャッシュ無効化（revalidateTag + webapp API呼び出し）

現在の実装:
```typescript
export const loadTransactionsPageData = (params: GetTransactionsBySlugParams) => {
  const cacheKey = ["transactions-page-data", JSON.stringify(params)];
  return unstable_cache(
    async () => { /* ... */ },
    cacheKey,
    { revalidate: 3600 },
  )();
};
```

API化後は、Redisなどを使った本格的なキャッシュ戦略を検討します。

---

## CSV形式仕様

### インポート形式（マネーフォワードクラウド会計形式）

```csv
取引No,取引日,決算整理仕訳,借方勘定科目,借方補助科目,借方部門,借方取引先,借方税区分,借方金額,貸方勘定科目,貸方補助科目,貸方部門,貸方取引先,貸方税区分,貸方金額,摘要
TX001,2024/08/15,いいえ,人件費,給料手当,本部,事務所A,対象外,300000,現金預金,普通預金,本部,銀行A,対象外,300000,事務所スタッフ給与
```

### エクスポート形式（簡易版）

```csv
日付,年月,種別,カテゴリ,サブカテゴリ,アカウント,ラベル,金額
2024-08-15,2024.08,支出,人件費,給料手当,人件費,事務所スタッフ給与,-300000
```
