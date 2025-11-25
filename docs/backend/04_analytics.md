# データ集計API

トランザクションデータを集計し、可視化用のデータを提供するAPIエンドポイント。

---

## エンドポイント一覧

### 1. トップページデータ取得

トップページに必要な全データ（トランザクション一覧、月次集計、サンキー図、貸借対照表）を一括取得します（公開API）。

**エンドポイント**
```
GET /api/analytics/top-page
```

**認証**: 不要

**クエリパラメータ**
```typescript
{
  slugs: string[];           // 必須: 政治団体のSlug（複数指定可）
  financialYear: number;     // 必須: 会計年度
  page?: number;             // ページ番号（デフォルト: 1）
  perPage?: number;          // 1ページあたりの件数（デフォルト: 50）
}
```

**リクエスト例**
```
GET /api/analytics/top-page?slugs=yamada-taro&financialYear=2024&page=1&perPage=50
```

**レスポンス**
```typescript
{
  transactionData: {
    transactions: DisplayTransaction[];
    total: number;
    page: number;
    perPage: number;
    totalPages: number;
    politicalOrganizations: PoliticalOrganization[];
    lastUpdatedAt: string | null;
  };
  monthlyData: MonthlyAggregation[];
  political: SankeyData;        // 政治資金規正法カテゴリ
  friendly: SankeyData;         // フレンドリーカテゴリ
  balanceSheetData: BalanceSheetData;
}
```

**レスポンス例**
```json
{
  "transactionData": {
    "transactions": [/* トランザクション一覧 */],
    "total": 1523,
    "page": 1,
    "perPage": 50,
    "totalPages": 31,
    "politicalOrganizations": [/* 政治団体情報 */],
    "lastUpdatedAt": "2024-08-20T10:30:00.000Z"
  },
  "monthlyData": [
    {
      "yearMonth": "2024-01",
      "income": 5000000,
      "expense": 4000000
    }
  ],
  "political": {
    "nodes": [/* サンキー図ノード */],
    "links": [/* サンキー図リンク */],
    "totalLatestBalance": 10000000
  },
  "friendly": {
    "nodes": [/* サンキー図ノード */],
    "links": [/* サンキー図リンク */],
    "totalLatestBalance": 10000000
  },
  "balanceSheetData": {
    "left": {
      "currentAssets": 10000000,
      "fixedAssets": 0,
      "debtExcess": 0
    },
    "right": {
      "currentLiabilities": 500000,
      "fixedLiabilities": 2000000,
      "netAssets": 7500000
    }
  }
}
```

**ステータスコード**
- `200 OK`: 成功
- `400 Bad Request`: パラメータエラー
- `404 Not Found`: 指定されたSlugの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

**特徴**
- 5つのUsecaseを段階的に並列実行（データベースコネクションプール考慮）
- 第1段階: transaction、monthly、balanceSheetを並列実行
- 第2段階: sankeyの2種類（political-category、friendly-category）を並列実行

---

### 2. 月次収支集計取得

月ごとの収入・支出を集計します（公開API）。

**エンドポイント**
```
GET /api/analytics/monthly
```

**認証**: 不要

**クエリパラメータ**
```typescript
{
  slugs: string[];           // 必須: 政治団体のSlug（複数指定可）
  financialYear: number;     // 必須: 会計年度
}
```

**リクエスト例**
```
GET /api/analytics/monthly?slugs=yamada-taro&financialYear=2024
```

**レスポンス**
```typescript
{
  monthlyData: MonthlyAggregation[];
}
```

**レスポンス例**
```json
{
  "monthlyData": [
    {
      "yearMonth": "2024-01",
      "income": 5000000,
      "expense": 4000000
    },
    {
      "yearMonth": "2024-02",
      "income": 4500000,
      "expense": 4200000
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
- 月次収支グラフの表示
- 収支トレンドの分析

---

### 3. サンキー図データ取得

収入から支出への資金フローをサンキー図用に集計します（公開API）。

**エンドポイント**
```
GET /api/analytics/sankey
```

**認証**: 不要

**クエリパラメータ**
```typescript
{
  slugs: string[];                           // 必須: 政治団体のSlug（複数指定可）
  financialYear: number;                     // 必須: 会計年度
  categoryType?: "political-category" | "friendly-category"; // カテゴリタイプ（デフォルト: political-category）
}
```

**リクエスト例**
```
GET /api/analytics/sankey?slugs=yamada-taro&financialYear=2024&categoryType=friendly-category
```

**レスポンス**
```typescript
{
  sankeyData: SankeyData;
}
```

**レスポンス例**
```json
{
  "sankeyData": {
    "nodes": [
      {
        "id": "income_donation",
        "label": "寄付金",
        "nodeType": "income"
      },
      {
        "id": "total",
        "label": "合計",
        "nodeType": "total"
      },
      {
        "id": "expense_personnel",
        "label": "人件費",
        "nodeType": "expense"
      }
    ],
    "links": [
      {
        "source": "income_donation",
        "target": "total",
        "value": 5000000
      },
      {
        "source": "total",
        "target": "expense_personnel",
        "value": 3000000
      }
    ],
    "totalLatestBalance": 10000000
  }
}
```

**ステータスコード**
- `200 OK`: 成功
- `400 Bad Request`: パラメータエラー
- `404 Not Found`: 指定されたSlugの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

**カテゴリタイプ**
- `political-category`: 政治資金規正法に基づくカテゴリ（デフォルト）
- `friendly-category`: 一般向けにわかりやすく分類したカテゴリ

**用途**
- サンキー図による資金フロー可視化
- カテゴリ別の収支バランス表示

---

### 4. 貸借対照表データ取得

会計年度末時点での資産・負債・純資産を集計します（公開API）。

**エンドポイント**
```
GET /api/analytics/balance-sheet
```

**認証**: 不要

**クエリパラメータ**
```typescript
{
  slugs: string[];           // 必須: 政治団体のSlug（複数指定可）
  financialYear: number;     // 必須: 会計年度
}
```

**リクエスト例**
```
GET /api/analytics/balance-sheet?slugs=yamada-taro&financialYear=2024
```

**レスポンス**
```typescript
{
  balanceSheetData: BalanceSheetData;
}
```

**レスポンス例**
```json
{
  "balanceSheetData": {
    "left": {
      "currentAssets": 10000000,
      "fixedAssets": 0,
      "debtExcess": 0
    },
    "right": {
      "currentLiabilities": 500000,
      "fixedLiabilities": 2000000,
      "netAssets": 7500000
    }
  }
}
```

**ステータスコード**
- `200 OK`: 成功
- `400 Bad Request`: パラメータエラー
- `404 Not Found`: 指定されたSlugの政治団体が存在しない
- `500 Internal Server Error`: サーバーエラー

**計算ロジック**
- **流動資産**: 最新残高スナップショットの合計
- **固定資産**: 現状は0（固定資産なし）
- **流動負債**: 未払金の残高（未払金収入 - 未払金支出）
- **固定負債**: 借入金の残高（借入金収入 - 借入金返済）
- **純資産**: (流動資産 + 固定資産) - (流動負債 + 固定負債)
  - 正の場合: 純資産として計上、債務超過は0
  - 負の場合: 純資産は0、債務超過として計上

**用途**
- 貸借対照表の表示
- 財務状況の健全性チェック

---

### 5. 日次寄付金推移データ取得

日ごとの寄付金額と累積額を集計します（公開API）。

**エンドポイント**
```
GET /api/analytics/daily-donations
```

**認証**: 不要

**クエリパラメータ**
```typescript
{
  slugs: string[];           // 必須: 政治団体のSlug（複数指定可）
  financialYear: number;     // 必須: 会計年度
}
```

**リクエスト例**
```
GET /api/analytics/daily-donations?slugs=yamada-taro&financialYear=2024
```

**レスポンス**
```typescript
{
  dailyDonations: DailyDonationData[];
}
```

**レスポンス例**
```json
{
  "dailyDonations": [
    {
      "date": "2024-01-01",
      "dailyAmount": 100000,
      "cumulativeAmount": 100000
    },
    {
      "date": "2024-01-02",
      "dailyAmount": 50000,
      "cumulativeAmount": 150000
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
- 寄付金推移グラフの表示
- 寄付キャンペーンの効果測定

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

---

## 使用例（フロントエンド）

### トップページデータ取得
```typescript
const params = new URLSearchParams({
  slugs: 'yamada-taro',
  financialYear: '2024',
  page: '1',
  perPage: '50'
});
const response = await fetch(`/api/analytics/top-page?${params}`);
const {
  transactionData,
  monthlyData,
  political,
  friendly,
  balanceSheetData
} = await response.json();
```

### 月次収支集計取得
```typescript
const params = new URLSearchParams({
  slugs: 'yamada-taro',
  financialYear: '2024'
});
const response = await fetch(`/api/analytics/monthly?${params}`);
const { monthlyData } = await response.json();
```

### サンキー図データ取得
```typescript
const params = new URLSearchParams({
  slugs: 'yamada-taro',
  financialYear: '2024',
  categoryType: 'friendly-category'
});
const response = await fetch(`/api/analytics/sankey?${params}`);
const { sankeyData } = await response.json();
```

### 貸借対照表データ取得
```typescript
const params = new URLSearchParams({
  slugs: 'yamada-taro',
  financialYear: '2024'
});
const response = await fetch(`/api/analytics/balance-sheet?${params}`);
const { balanceSheetData } = await response.json();
```

### 日次寄付金推移データ取得
```typescript
const params = new URLSearchParams({
  slugs: 'yamada-taro',
  financialYear: '2024'
});
const response = await fetch(`/api/analytics/daily-donations?${params}`);
const { dailyDonations } = await response.json();
```

---

## データベース操作

現在の実装では、以下のUsecaseとRepositoryを使用しています：

**Usecase**
- `GetTransactionsBySlugUsecase`: トランザクション一覧
- `GetMonthlyTransactionAggregationUsecase`: 月次集計
- `GetSankeyAggregationUsecase`: サンキー図データ
- `GetBalanceSheetUsecase`: 貸借対照表
- `GetDailyDonationUsecase`: 日次寄付金推移

**Repository**
- `PrismaTransactionRepository`
  - `getMonthlyAggregation()`: 月次集計
  - `getCategoryAggregationForSankey()`: カテゴリ別集計（サンキー図用）
  - `getDailyDonationData()`: 日次寄付金集計
  - `getBorrowingIncomeTotal()`: 借入金収入合計
  - `getBorrowingExpenseTotal()`: 借入金返済合計
  - `getLiabilityBalance()`: 未払金残高
- `PrismaBalanceSnapshotRepository`
  - `getTotalLatestBalanceByOrgIds()`: 最新残高合計
  - `getTotalLatestBalancesByYear()`: 会計年度別最新残高合計

---

## キャッシュ戦略

### トップページデータ
- **キャッシュ時間**: 3600秒（1時間）
- **キャッシュキー**: `["top-page-data"]` + パラメータのハッシュ
- **無効化タイミング**: トランザクション登録/削除時

### 個別エンドポイント
- **月次集計**: 3600秒キャッシュ
- **サンキー図**: 3600秒キャッシュ
- **貸借対照表**: 3600秒キャッシュ
- **日次寄付金**: 3600秒キャッシュ

現在の実装（Next.js unstable_cache）:
```typescript
export const loadTopPageData = unstable_cache(
  async (params: TopPageDataParams) => {
    // ... データ取得処理
  },
  ["top-page-data"],
  { revalidate: 3600 },
);
```

API化後の推奨構成:
- **Redis**: キャッシュストア
- **Cache-Control ヘッダー**: ブラウザキャッシュ制御
- **ETag**: 条件付きリクエスト対応
- **キャッシュ無効化API**: 管理画面からのキャッシュクリア機能

---

## パフォーマンス最適化

### データベースクエリ最適化
- IN句を使った複数組織の一括取得
- インデックスの活用（politicalOrganizationId, financialYear, transactionType, transactionDate）
- 不要なJOINの削減

### 並列処理
- 独立したデータ取得は並列実行
- データベースコネクションプール枯渇を避けるため段階的に実行

### レスポンスサイズ削減
- ページネーション必須
- 不要なフィールドの除外
- gzip圧縮

### 計算結果のキャッシュ
- 集計結果を事前計算してキャッシュ
- バックグラウンドジョブでの定期更新検討

---

## モックデータ対応

開発・テスト用にモックデータ機能をサポートします。

**環境変数**
```
USE_MOCK_DATA=true
```

**モックデータ提供範囲**
- トップページデータ（全データ）
- トランザクション一覧
- 月次集計
- サンキー図データ
- 貸借対照表

**実装**
```typescript
if (process.env.USE_MOCK_DATA === "true") {
  const mockUsecase = new GetMockTransactionPageDataUsecase();
  return await mockUsecase.execute(params);
}
```
