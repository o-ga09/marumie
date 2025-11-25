# 不要機能の詳細分析

このドキュメントでは、政治資金システムから家計簿システムへの転用において、削除・変更が必要な機能を詳細に分析します。

---

## 1. 公開ページ（webapp）の削除

### 現状の構成

```
webapp/
├── src/
│   ├── app/              # Next.js App Router
│   │   ├── page.tsx      # トップページ（政治家一覧）
│   │   ├── politicians/  # 政治家詳細ページ
│   │   └── api/          # API Routes
│   ├── client/           # クライアントコンポーネント
│   ├── server/           # サーバーサイドロジック
│   └── types/
└── public/
```

### 削除対象ファイル

#### 完全削除
```
webapp/src/app/
├── page.tsx                                    # トップページ
├── politicians/
│   └── [slug]/
│       └── page.tsx                            # 政治家詳細ページ
└── layout.tsx（公開用レイアウト）               # 公開用ヘッダー・フッター
```

#### クライアントコンポーネント（公開用）
```
webapp/src/client/components/
├── OrganizationList.tsx                        # 政治団体一覧
├── TransactionTable.tsx（公開用のみ）           # 公開用トランザクション表
├── PublicHeader.tsx                            # 公開用ヘッダー
├── PublicFooter.tsx                            # 公開用フッター
└── ShareButton.tsx                             # 共有ボタン
```

### 移行・統合対象

以下は admin に移行して活用：
```
webapp/src/client/components/
├── MonthlyChart.tsx                            # ✅ 月次グラフ → 家計ダッシュボードで使用
├── SankeyDiagram.tsx                           # ✅ サンキー図 → キャッシュフロー分析で使用
├── BalanceSheetView.tsx                        # ✅ 貸借対照表 → 資産負債表示で使用
└── TransactionFilters.tsx                      # ✅ フィルタ → トランザクション管理で使用
```

---

## 2. Slug（公開用ID）の削除

### 現状の使用箇所

#### データモデル
```typescript
// prisma/schema.prisma
model PoliticalOrganization {
  slug   String   @unique @db.VarChar(255)  // ❌ 削除
  // ...
}
```

#### API エンドポイント
```typescript
// webapp/src/server/loaders/
GET /api/political-organizations/:slug        // ❌ 削除
GET /api/transactions?slugs=yamada-taro       // ❌ slugs パラメータ削除
```

#### URL 構造
```
https://example.com/politicians/yamada-taro   // ❌ 削除
↓
https://example.com/households/uuid           // ✅ UUID使用（認証必須）
```

### 変更対応

1. **データベース**: `slug` カラムを削除
2. **API**: すべてのエンドポイントで `id` (UUID) を使用
3. **URL**: UUID ベースのプライベートURL

---

## 3. 複数団体の横断表示

### 現状の機能

```typescript
// 複数の政治団体を同時に表示
GET /api/transactions?slugs=yamada-taro&slugs=tanaka-hanako&financialYear=2024

// 複数団体の合算集計
const organizations = await politicalOrganizationRepository.findBySlugs(params.slugs);
const organizationIds = organizations.map((org) => org.id);
```

### 削除理由

- 家計簿では基本的に1ユーザー1家計
- 複数家計を持つケースは稀（ビジネス用と個人用など）
- 複雑性が増す割にメリットが少ない

### 代替案

1. **シンプルな実装**: 1ユーザー = 1家計
2. **複数家計サポート（将来）**: 
   - ユーザーが複数の家計に所属可能
   - 家計を切り替えて使用（同時表示はしない）

---

## 4. 政治資金規正法固有のカテゴリ

### 削除対象のカテゴリ体系

#### 収入カテゴリ
```typescript
// 政治資金規正法カテゴリ
const INCOME_CATEGORIES = {
  'income_donation': '寄付金',
  'income_party_grant': '政党交付金',
  'income_membership_fee': '会費',
  'income_business': '事業収入',
  'income_borrowing': '借入金',
  'income_other': 'その他収入'
};

// ❌ 削除：政治資金特有
```

#### 支出カテゴリ
```typescript
// 政治資金規正法カテゴリ
const EXPENSE_CATEGORIES = {
  'expense_personnel': '人件費',
  'expense_office': '事務所費',
  'expense_publicity': '宣伝広告費',
  'expense_political_activity': '政治活動費',
  'expense_organization': '組織活動費',
  'expense_election': '選挙関係費',
  'expense_borrowing_repayment': '借入金返済',
  'expense_other': 'その他支出'
};

// ❌ 削除：政治資金特有
```

### 新しいカテゴリ体系

#### 家計簿用デフォルトカテゴリ
```typescript
// 収入カテゴリ
const HOUSEHOLD_INCOME_CATEGORIES = {
  'income_salary': '給与・賞与',
  'income_side_business': '副業収入',
  'income_investment': '投資収益',
  'income_bonus': '臨時収入',
  'income_allowance': '手当・補助',
  'income_gift': '贈与',
  'income_other': 'その他収入'
};

// 支出カテゴリ
const HOUSEHOLD_EXPENSE_CATEGORIES = {
  // 固定費
  'expense_housing': '住居費',
  'expense_utilities': '光熱費',
  'expense_communication': '通信費',
  'expense_insurance': '保険料',
  'expense_subscription': 'サブスクリプション',
  
  // 変動費
  'expense_food': '食費',
  'expense_daily_goods': '日用品',
  'expense_clothing': '被服費',
  'expense_medical': '医療費',
  'expense_transportation': '交通費',
  'expense_entertainment': '娯楽・交際費',
  'expense_education': '教育費',
  'expense_beauty': '美容費',
  
  // その他
  'expense_tax': '税金',
  'expense_saving': '貯蓄・投資',
  'expense_loan_repayment': 'ローン返済',
  'expense_other': 'その他支出'
};
```

---

## 5. CSV形式の変更

### 現状（マネーフォワードクラウド会計形式）

```csv
取引No,取引日,決算整理仕訳,借方勘定科目,借方補助科目,借方部門,借方取引先,借方税区分,借方金額,貸方勘定科目,貸方補助科目,貸方部門,貸方取引先,貸方税区分,貸方金額,摘要
TX001,2024/08/15,いいえ,人件費,給料手当,本部,事務所A,対象外,300000,現金預金,普通預金,本部,銀行A,対象外,300000,事務所スタッフ給与
```

### 変更後（家計簿形式）

```csv
日付,金額,カテゴリ,サブカテゴリ,アカウント,メモ,タグ
2024-08-15,-50000,食費,外食,クレジットカードA,会社の同僚と飲み会,#会社 #飲み会
2024-08-20,300000,給与・賞与,給与,銀行口座A,8月分給与,
2024-08-25,-15000,光熱費,電気代,銀行口座A,8月分電気代,
```

### 削除する複式簿記関連フィールド

- ❌ 借方勘定科目、借方補助科目、借方部門、借方取引先、借方税区分
- ❌ 貸方勘定科目、貸方補助科目、貸方部門、貸方取引先、貸方税区分
- ❌ 決算整理仕訳

### 追加するフィールド

- ✅ アカウント（銀行口座、クレジットカード等）
- ✅ タグ（複数可）

---

## 6. 貸借対照表の簡素化

### 現状（政治資金向け）

```typescript
interface BalanceSheetData {
  left: {
    currentAssets: number;      // 流動資産
    fixedAssets: number;        // 固定資産
    debtExcess: number;         // 債務超過
  };
  right: {
    currentLiabilities: number; // 流動負債（未払金）
    fixedLiabilities: number;   // 固定負債（借入金）
    netAssets: number;          // 純資産
  };
}
```

### 変更後（家計簿向け）

```typescript
interface NetWorthData {
  assets: {
    cash: number;               // 現金・預金
    investment: number;         // 投資資産
    realEstate: number;         // 不動産
    other: number;              // その他資産
    total: number;              // 資産合計
  };
  liabilities: {
    mortgage: number;           // 住宅ローン
    carLoan: number;            // 自動車ローン
    creditCard: number;         // クレジットカード債務
    other: number;              // その他負債
    total: number;              // 負債合計
  };
  netWorth: number;             // 純資産（資産 - 負債）
}
```

### 簡素化のポイント

- ✅ 流動/固定の区別を削除（家計では不要）
- ✅ より直感的な分類（現金、投資、不動産など）
- ❌ 債務超過の概念を削除（ネガティブな純資産として表示）

---

## 7. 会計年度の柔軟化

### 現状（固定）

```typescript
// 会計年度は1月〜12月固定
financialYear: 2024  // 2024年1月1日〜2024年12月31日
```

### 変更後（ユーザー設定可能）

```typescript
// ユーザーが会計年度の開始月を設定可能
interface Household {
  fiscalYearStart: number;  // 1-12（デフォルト: 1）
}

// 例：4月開始の場合
// fiscalYearStart: 4
// fiscalYear: 2024 → 2024年4月1日〜2025年3月31日
```

### 変更箇所

```typescript
// utils/financial-year.ts
export function getFinancialYearRange(year: number, startMonth: number = 1) {
  const startDate = new Date(year, startMonth - 1, 1);
  const endDate = new Date(year + 1, startMonth - 1, 0); // 前日
  return { startDate, endDate };
}
```

---

## 8. サンキー図のカスタマイズ

### 現状（政治資金向け）

```
収入カテゴリ → 合計 → 支出カテゴリ

[寄付金]   ─────┐
[政党交付金] ────┤
[会費]     ─────┤→ [合計] → [人件費]
[事業収入] ─────┤         → [事務所費]
[その他]   ─────┘         → [宣伝広告費]
                          → [その他]
```

### 変更後（家計簿向け）

```
収入 → アカウント → 支出

[給与]     ────┐
[副業]     ────┤→ [銀行口座] → [食費]
[投資収益] ────┤  [現金]     → [住居費]
               ┘  [カード]   → [光熱費]
                              → [娯楽費]
```

**変更点**:
- 中間に「アカウント」ノードを追加
- より直感的な資金フロー表示

---

## 9. 認証不要APIの廃止

### 削除対象エンドポイント（認証不要版）

```typescript
// 公開API（現状）
GET /api/political-organizations              // ❌
GET /api/political-organizations/:slug        // ❌
GET /api/transactions                         // ❌
GET /api/transactions/:id                     // ❌
GET /api/transactions/export/csv              // ❌
GET /api/analytics/top-page                   // ❌
GET /api/analytics/monthly                    // ❌
GET /api/analytics/sankey                     // ❌
GET /api/analytics/balance-sheet              // ❌
GET /api/analytics/daily-donations            // ❌
GET /api/balance-snapshots/latest             // ❌
```

### すべて認証必須化

```typescript
// 家計簿API（すべて認証必須）
GET /api/households                           // ✅ 自分の家計のみ
GET /api/households/:id                       // ✅ アクセス権確認
GET /api/transactions                         // ✅ 自分の家計のみ
// ... 以下すべて認証必須
```

---

## 10. 用語の変更

### 置換が必要な用語

| 旧用語（政治資金） | 新用語（家計簿） | 置換対象 |
|-------------------|-----------------|---------|
| PoliticalOrganization | Household | コード全体 |
| political_organization_id | household_id | データベース・コード |
| slug | （削除） | データベース・コード |
| 政治団体 | 家計 | UI・ドキュメント |
| 会計年度 | 年度 | UI |
| 寄付金 | （カテゴリ変更） | カテゴリマスタ |
| 政党交付金 | （カテゴリ変更） | カテゴリマスタ |
| 借入金 | ローン | UI・カテゴリマスタ |
| displayName | name | データベース・コード |
| orgName | （削除） | データベース |

---

## 11. 不要なファイル・ディレクトリ一覧

### 完全削除対象

```
webapp/                                        # 公開ページ全体
├── src/app/page.tsx
├── src/app/politicians/
├── src/client/components/
│   ├── OrganizationList.tsx
│   ├── PublicHeader.tsx
│   ├── PublicFooter.tsx
│   └── ShareButton.tsx
└── public/
    └── og-images/（政治家の画像等）

docs/
├── 20250815_1434_政治とカネダッシュボード*.md  # 政治資金特有のドキュメント
└── images/（政治資金関連の図）

shared/utils/
└── political-category-mapping.ts              # 政治資金カテゴリマッピング
```

### 変更が必要なファイル

```
prisma/schema.prisma                           # データモデル全体
prisma/seed.cjs                                # シードデータ

admin/src/
├── app/                                       # ルート構造変更
├── server/                                    # 認証・認可の追加
└── client/components/                         # UI用語変更

shared/
├── models/                                    # モデル名変更
└── utils/                                     # ユーティリティ関数の見直し

*.md                                           # ドキュメント全体の用語変更
```

---

## 12. データベーススキーマ変更サマリー

### 削除するカラム

```sql
-- PoliticalOrganization（Householdに変更）
ALTER TABLE political_organizations DROP COLUMN slug;
ALTER TABLE political_organizations DROP COLUMN org_name;

-- Transaction
-- （特になし、カテゴリ体系の変更はアプリケーション層で対応）
```

### 追加するテーブル

```sql
-- 家計メンバー
CREATE TABLE household_members (
  id UUID PRIMARY KEY,
  household_id UUID REFERENCES households(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role VARCHAR(20) NOT NULL DEFAULT 'viewer',
  joined_at TIMESTAMP NOT NULL DEFAULT NOW(),
  UNIQUE(household_id, user_id)
);

-- カテゴリマスタ
CREATE TABLE categories (
  id UUID PRIMARY KEY,
  household_id UUID REFERENCES households(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  type VARCHAR(20) NOT NULL,
  parent_id UUID REFERENCES categories(id),
  color VARCHAR(20),
  icon VARCHAR(50),
  is_default BOOLEAN DEFAULT FALSE,
  is_active BOOLEAN DEFAULT TRUE,
  display_order INT DEFAULT 0,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- 予算
CREATE TABLE budgets (
  id UUID PRIMARY KEY,
  household_id UUID REFERENCES households(id) ON DELETE CASCADE,
  category_id UUID REFERENCES categories(id) ON DELETE CASCADE,
  year INT NOT NULL,
  month INT NOT NULL,
  amount DECIMAL(15,2) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  UNIQUE(household_id, category_id, year, month)
);

-- アカウント
CREATE TABLE accounts (
  id UUID PRIMARY KEY,
  household_id UUID REFERENCES households(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  type VARCHAR(20) NOT NULL,
  initial_balance DECIMAL(15,2) DEFAULT 0,
  is_active BOOLEAN DEFAULT TRUE,
  display_order INT DEFAULT 0,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- タグ
CREATE TABLE tags (
  id UUID PRIMARY KEY,
  household_id UUID,
  name VARCHAR(255) NOT NULL,
  color VARCHAR(20),
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- トランザクションタグ
CREATE TABLE transaction_tags (
  transaction_id BIGINT REFERENCES transactions(id) ON DELETE CASCADE,
  tag_id UUID REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY(transaction_id, tag_id)
);
```

---

## 13. マイグレーション手順

### Step 1: データベース変更

```bash
# 1. バックアップ
pg_dump marumie > backup_before_migration.sql

# 2. 新しいテーブル追加
npx prisma migrate dev --name add_household_budget_tables

# 3. データ移行
# - political_organizations → households
# - slug 削除
# - デフォルトカテゴリの登録

# 4. 動作確認
```

### Step 2: コード変更

```bash
# 1. webapp ディレクトリを削除または非活性化
mv webapp webapp_old

# 2. admin を app にリネーム
mv admin app

# 3. 用語の一括置換
# PoliticalOrganization → Household
# political_organization → household
# slug → id（UUIDへ）

# 4. 不要なファイル削除
```

### Step 3: 機能追加

```bash
# 1. カテゴリ管理機能
# 2. 予算管理機能
# 3. アカウント管理機能
# 4. 家族共有機能（オプション）
```

---

## まとめ

### 削除ボリューム

- **ファイル削除**: 約30-40ファイル（webapp全体）
- **コード削除**: 約5,000-7,000行
- **テーブル削除**: なし（既存テーブルは活用）

### 変更ボリューム

- **ファイル変更**: 約100-150ファイル（用語変更）
- **コード変更**: 約10,000-15,000行
- **テーブル追加**: 6テーブル
- **カラム追加**: 約10-15カラム

### 開発期間見積もり

- **Phase 1（基盤整備）**: 1-2週間
- **Phase 2（コア機能）**: 2-3週間
- **Phase 3（UI/UX）**: 2-3週間
- **合計**: 5-8週間

### リスク評価

- **低リスク**: データベース構造は大部分を活用可能
- **中リスク**: 認証・認可の全体適用
- **高リスク**: マルチテナント対応（データ隔離）
