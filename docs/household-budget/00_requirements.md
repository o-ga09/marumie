# 家計簿可視化プラットフォーム - 要件定義

## プロジェクト概要

### 目的
政治資金可視化プラットフォームのアーキテクチャを活用し、家計簿の可視化プラットフォームに転用する。

### ターゲットユーザー
- **個人ユーザー**: 自分の家計を管理したい個人
- **家族ユーザー**: 家族全体の家計を共同管理したいグループ
- **アドバイザー**: ファイナンシャルプランナーなどの専門家（将来的に）

---

## 現行システムとの対応関係

### 概念マッピング

| 政治資金システム | 家計簿システム | 説明 |
|-----------------|---------------|------|
| 政治団体（PoliticalOrganization） | 家計（Household） | 管理単位 |
| 取引（Transaction） | 取引（Transaction） | 収入・支出記録 |
| 収入カテゴリ（寄付金、政党交付金等） | 収入カテゴリ（給与、副業、投資等） | 収入の分類 |
| 支出カテゴリ（人件費、事務所費等） | 支出カテゴリ（食費、住居費、光熱費等） | 支出の分類 |
| 会計年度（FinancialYear） | 年度（Year） | 集計期間 |
| 残高スナップショット | 残高スナップショット | 資産状況の記録 |
| 貸借対照表 | 資産負債表 / 純資産 | 財務状況の可視化 |
| サンキー図（資金フロー） | サンキー図（キャッシュフロー） | お金の流れの可視化 |
| 月次収支グラフ | 月次収支グラフ | 月ごとの収支推移 |
| 寄付金推移 | 投資・貯蓄推移（オプション） | 特定カテゴリの推移 |
| CSV一括登録 | CSV一括登録 | マネーフォワード等からのインポート |
| 管理画面（admin） | 個人管理画面（dashboard） | データ管理UI |
| 公開ページ（webapp） | **不要** or **家族共有ページ** | 透明性の要否 |

---

## 主要な変更点

### 1. 公開性の変更
**政治資金**: 透明性が目的 → 一般公開が前提
**家計簿**: プライバシーが重要 → 非公開が前提

**対応**:
- webappの公開機能は**削除**または**家族共有機能**に変更
- 認証必須化（全エンドポイント）
- ユーザー単位でのデータ隔離

### 2. マルチテナント対応
**政治資金**: 複数の政治団体を独立管理
**家計簿**: 複数のユーザーがそれぞれの家計を独立管理

**対応**:
- ユーザーと家計の紐付け強化
- データアクセス制御の厳格化
- 家族共有機能（複数ユーザーで1つの家計を共同管理）

### 3. カテゴリ体系の変更
**政治資金**: 政治資金規正法に基づく固定カテゴリ
**家計簿**: ユーザーが自由にカスタマイズ可能なカテゴリ

**対応**:
- カテゴリマスタテーブルの追加
- ユーザー定義カテゴリのサポート
- デフォルトカテゴリテンプレートの提供

### 4. 会計年度の概念
**政治資金**: 会計年度（1月〜12月）で管理
**家計簿**: 年度の概念は任意（月次管理が主）

**対応**:
- 年度の概念は維持（統計に便利）
- 開始月の設定を可能に（4月開始など）
- 月次・週次・日次集計の強化

### 5. 透明性機能の削除
**政治資金**: 公開用の詳細表示、CSV公開エクスポート
**家計簿**: プライベートデータ、外部公開不要

**対応**:
- 公開用エンドポイントの削除または認証必須化
- Slug（URLフレンドリーID）の削除検討
- 共有機能は家族内のみ（オプション）

---

## 必須機能

### ✅ そのまま活用できる機能

1. **トランザクション管理**
   - 収入・支出の記録
   - 日付、金額、カテゴリ、メモの管理
   - CSV一括インポート
   - フィルタリング・ソート・ページネーション

2. **データ集計・可視化**
   - 月次収支グラフ
   - カテゴリ別集計
   - サンキー図（キャッシュフロー）
   - 残高推移

3. **残高管理**
   - 残高スナップショット
   - 資産・負債の管理
   - 純資産の計算

4. **CSV連携**
   - マネーフォワードクラウド形式の取込
   - CSVプレビュー機能
   - 重複チェック

5. **データベース設計**
   - Prismaスキーマ
   - トランザクション管理の仕組み
   - インデックス設計

---

## 削除・変更が必要な機能

### ❌ 削除すべき機能

1. **公開ページ（webapp）**
   - 一般公開用のフロントエンド
   - Slug（URLフレンドリーID）による公開アクセス
   - 認証不要のAPIエンドポイント

   **代替案**: 
   - 家族共有機能（招待制）
   - または完全削除してadminのみに統合

2. **複数団体の横断表示**
   - 複数の政治団体を同時に表示する機能
   - slugs配列による複数指定

   **代替案**: 
   - 基本は1ユーザー1家計
   - 家族共有で複数ユーザーが同じ家計にアクセス

3. **政治資金規正法固有のカテゴリ**
   - 寄付金、政党交付金、借入金などの固定カテゴリ
   - 政治資金用の勘定科目体系

   **代替案**: 
   - 家計簿用カテゴリテンプレート
   - ユーザー定義カテゴリ

4. **CSVエクスポート（公開用）**
   - 認証不要のCSVダウンロード

   **代替案**: 
   - 認証必須のCSVエクスポート（自分のデータのみ）

### 🔄 変更が必要な機能

1. **認証・認可**
   - **現状**: 管理画面のみ認証必須、公開ページは認証不要
   - **変更後**: すべて認証必須、ユーザー単位でデータ隔離

2. **データモデル**
   - **PoliticalOrganization → Household**
   - **User（管理者用） → User（一般ユーザー）**
   - **slug（公開用ID） → 削除または内部ID化**

3. **カテゴリ管理**
   - **現状**: アプリケーション内で固定
   - **変更後**: データベースでマスタ管理、ユーザーがカスタマイズ可能

4. **会計年度**
   - **現状**: 1月〜12月固定
   - **変更後**: ユーザーが開始月を設定可能（デフォルト1月）

---

## 新規追加が必要な機能

### ➕ 家計簿特有の機能

1. **予算管理**
   - カテゴリ別の月次予算設定
   - 予算vs実績の比較
   - 予算超過アラート

2. **定期支出・収入**
   - 家賃、光熱費などの定期支出を登録
   - 給与などの定期収入を登録
   - 自動入力支援

3. **タグ機能**
   - トランザクションに複数のタグを付与
   - タグ別の集計・フィルタリング
   - 例: #外食 #デート #出張 など

4. **複数アカウント管理**
   - 銀行口座、クレジットカード、現金を個別管理
   - アカウント間の振替記録
   - アカウント別の残高管理

5. **家族共有機能**
   - 同じ家計を複数ユーザーで共同管理
   - ユーザーごとの権限設定（閲覧のみ / 編集可能）
   - 家族メンバーの招待機能

6. **レシート撮影・OCR（将来）**
   - スマホでレシートを撮影
   - OCRで金額・品目を自動抽出
   - トランザクションに自動変換

7. **目標設定**
   - 貯蓄目標の設定
   - 達成率の表示
   - 目標達成までのシミュレーション

8. **レポート機能**
   - 月次レポート自動生成
   - 前月比較
   - PDFエクスポート

9. **通知機能**
   - 予算超過通知
   - 定期支払いのリマインダー
   - 月次レポートの自動送信

---

## データモデルの変更案

### 新規テーブル

```prisma
// 家計（旧: PoliticalOrganization）
model Household {
  id               String   @id @default(uuid())
  name             String
  description      String?
  fiscalYearStart  Int      @default(1)  // 会計年度開始月（1-12）
  currency         String   @default("JPY")
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt
  
  // Relations
  members          HouseholdMember[]
  transactions     Transaction[]
  balanceSnapshots BalanceSnapshot[]
  categories       Category[]
  budgets          Budget[]
  accounts         Account[]
}

// 家計メンバー（家族共有用）
model HouseholdMember {
  id           String   @id @default(uuid())
  householdId  String
  userId       String
  role         MemberRole  @default(viewer)
  joinedAt     DateTime @default(now())
  
  household    Household @relation(fields: [householdId], references: [id], onDelete: Cascade)
  user         User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([householdId, userId])
}

enum MemberRole {
  owner     // 家計のオーナー（全権限）
  editor    // 編集可能
  viewer    // 閲覧のみ
}

// カテゴリマスタ
model Category {
  id           String   @id @default(uuid())
  householdId  String?  // nullの場合はシステムデフォルト
  name         String
  type         TransactionType
  parentId     String?  // 親カテゴリ（階層構造）
  color        String?  // UI表示用の色
  icon         String?  // UI表示用のアイコン
  isDefault    Boolean  @default(false)
  isActive     Boolean  @default(true)
  displayOrder Int      @default(0)
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
  
  household    Household? @relation(fields: [householdId], references: [id], onDelete: Cascade)
  parent       Category?  @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children     Category[] @relation("CategoryHierarchy")
  transactions Transaction[]
  budgets      Budget[]
}

// 予算
model Budget {
  id          String   @id @default(uuid())
  householdId String
  categoryId  String
  year        Int
  month       Int
  amount      Decimal  @db.Decimal(15, 2)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  household   Household @relation(fields: [householdId], references: [id], onDelete: Cascade)
  category    Category  @relation(fields: [categoryId], references: [id], onDelete: Cascade)
  
  @@unique([householdId, categoryId, year, month])
}

// アカウント（銀行口座、クレジットカード等）
model Account {
  id              String   @id @default(uuid())
  householdId     String
  name            String
  type            AccountType
  initialBalance  Decimal  @db.Decimal(15, 2) @default(0)
  isActive        Boolean  @default(true)
  displayOrder    Int      @default(0)
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  household       Household @relation(fields: [householdId], references: [id], onDelete: Cascade)
  transactions    Transaction[]
}

enum AccountType {
  cash           // 現金
  bank           // 銀行口座
  credit_card    // クレジットカード
  e_money        // 電子マネー
  other          // その他
}

// タグ
model Tag {
  id              String   @id @default(uuid())
  householdId     String?  // nullの場合はシステムデフォルト
  name            String
  color           String?
  createdAt       DateTime @default(now())
  
  transactions    TransactionTag[]
}

// トランザクションとタグの中間テーブル
model TransactionTag {
  transactionId   String
  tagId           String
  
  transaction     Transaction @relation(fields: [transactionId], references: [id], onDelete: Cascade)
  tag             Tag         @relation(fields: [tagId], references: [id], onDelete: Cascade)
  
  @@id([transactionId, tagId])
}
```

### 既存テーブルの変更

```prisma
// Transaction（変更点のみ）
model Transaction {
  // ... 既存フィールド ...
  
  // 追加・変更フィールド
  householdId         String           // 旧: politicalOrganizationId
  categoryId          String?          // カテゴリマスタへの参照（新規）
  accountId           String?          // アカウントへの参照（新規）
  tags                TransactionTag[] // タグとの関連（新規）
  receiptImageUrl     String?          // レシート画像URL（将来）
  isRecurring         Boolean @default(false)  // 定期取引フラグ（新規）
  recurringGroupId    String?          // 定期取引グループID（新規）
  
  // 削除フィールド
  // - slug（不要）
  // - 政治資金固有のフィールド
}

// User（変更点のみ）
model User {
  // ... 既存フィールド ...
  
  // 追加フィールド
  displayName         String?          // 表示名
  avatarUrl           String?          // アバター画像URL
  timezone            String @default("Asia/Tokyo")
  locale              String @default("ja")
  households          HouseholdMember[] // 所属家計
  
  // 削除フィールド
  // - role（家計ごとに権限が異なるため）
}
```

---

## 画面構成の変更

### 現状（政治資金システム）
```
webapp (公開)
├── トップページ（政治家一覧）
├── 政治家詳細ページ
│   ├── トランザクション一覧
│   ├── 月次収支グラフ
│   ├── サンキー図
│   └── 貸借対照表

admin (管理画面)
├── ログイン
├── 政治団体管理
├── トランザクション管理
│   ├── CSV一括登録
│   └── トランザクション一覧
└── 残高スナップショット管理
```

### 変更後（家計簿システム）
```
app (統合アプリ)
├── ログイン・サインアップ
├── ダッシュボード（家計サマリー）
│   ├── 今月の収支
│   ├── 予算vs実績
│   ├── 残高推移
│   └── カテゴリ別円グラフ
├── トランザクション
│   ├── 一覧表示
│   ├── 新規登録
│   ├── CSV一括登録
│   └── フィルタリング・検索
├── 予算管理
│   ├── 月次予算設定
│   └── 予算vs実績
├── レポート
│   ├── 月次レポート
│   ├── 年次レポート
│   └── カスタムレポート
├── 設定
│   ├── プロフィール
│   ├── カテゴリ管理
│   ├── アカウント管理
│   ├── 家族共有設定
│   └── データエクスポート
└── （オプション）家族共有ページ
    └── 招待された家族メンバー用の閲覧ページ
```

---

## API変更サマリー

### 削除するエンドポイント
- ❌ `GET /api/political-organizations` （公開用一覧）
- ❌ `GET /api/political-organizations/:slug` （公開用詳細）
- ❌ `GET /api/transactions` （認証不要版）
- ❌ `GET /api/transactions/export/csv` （認証不要版）
- ❌ `GET /api/analytics/*` （認証不要版）

### 変更するエンドポイント
- 🔄 すべてのエンドポイントを認証必須化
- 🔄 `political-organizations` → `households`
- 🔄 データアクセス制御（自分の家計のみ）

### 新規追加するエンドポイント
- ➕ `GET /api/categories` - カテゴリ一覧
- ➕ `POST /api/categories` - カテゴリ作成
- ➕ `GET /api/budgets` - 予算一覧
- ➕ `POST /api/budgets` - 予算設定
- ➕ `GET /api/accounts` - アカウント一覧
- ➕ `POST /api/accounts` - アカウント作成
- ➕ `GET /api/households/:id/members` - 家族メンバー一覧
- ➕ `POST /api/households/:id/invite` - 家族招待
- ➕ `GET /api/reports/monthly` - 月次レポート
- ➕ `GET /api/analytics/budget-vs-actual` - 予算vs実績

---

## 移行ステップ

### Phase 1: 基盤整備（1-2週間）
1. データモデルの設計・マイグレーション
2. 認証の全体適用
3. Household モデルへの移行
4. カテゴリマスタの実装

### Phase 2: コア機能実装（2-3週間）
1. トランザクション管理のHousehold対応
2. カテゴリ管理機能
3. 予算管理機能
4. アカウント管理機能

### Phase 3: UI/UX改善（2-3週間）
1. ダッシュボードの実装
2. レポート機能
3. 家族共有機能（オプション）
4. タグ機能

### Phase 4: 拡張機能（継続的）
1. 定期取引機能
2. レシート撮影・OCR
3. 通知機能
4. モバイルアプリ

---

## リスクと対策

### リスク1: データプライバシー
**リスク**: 家計データは極めてセンシティブ
**対策**: 
- 厳格な認証・認可
- データ暗号化
- アクセスログ
- GDPR/個人情報保護法対応

### リスク2: マルチテナント実装の複雑さ
**リスク**: ユーザー間のデータ漏洩
**対策**: 
- Row Level Security（RLS）の活用
- すべてのクエリにhouseholdIdフィルタ
- E2Eテストでのデータ隔離確認

### リスク3: 既存コードベースの理解
**リスク**: 政治資金特有のロジックが残る
**対策**: 
- 段階的なリファクタリング
- テストカバレッジの向上
- ドキュメント整備

---

## まとめ

### 活用できる資産
✅ トランザクション管理の仕組み
✅ CSV連携機能
✅ データ集計・可視化ロジック
✅ Prisma + PostgreSQLのインフラ
✅ Next.js + Reactのフロントエンド基盤

### 削除・変更が必要な部分
❌ 公開ページ（webapp）
❌ Slug（公開用ID）
❌ 政治資金固有のカテゴリ
🔄 認証・認可の全体適用
🔄 データモデルの変更

### 新規追加が必要な機能
➕ カテゴリマスタ管理
➕ 予算管理
➕ アカウント管理
➕ 家族共有機能
➕ タグ機能
