# マイクロサービスアーキテクチャ設計案

## 概要

家計簿可視化プラットフォームをマイクロサービスアーキテクチャで構築する設計案です。

---

## アーキテクチャ全体図

```
┌─────────────────────────────────────────────────────────────────┐
│                        クライアント層                              │
├─────────────────────────────────────────────────────────────────┤
│  Web App (Next.js)     │  Mobile App (React Native/Flutter)     │
└───────────┬─────────────┴──────────────────┬────────────────────┘
            │                                │
            ▼                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway (Kong/AWS API Gateway)          │
│  - ルーティング                                                    │
│  - 認証・認可（JWT検証）                                            │
│  - レート制限                                                      │
│  - ロギング・モニタリング                                           │
└───────────┬─────────────────────────────────────────────────────┘
            │
            ├─────────────────────────────────────────────────────┐
            │                                                     │
            ▼                                                     ▼
┌───────────────────────┐                          ┌──────────────────────┐
│  認証サービス           │                          │  ユーザーサービス      │
│  (Auth Service)        │◄────────────────────────►│  (User Service)      │
│                       │                          │                      │
│  - ユーザー認証        │                          │  - プロフィール管理    │
│  - JWT発行            │                          │  - 設定管理           │
│  - パスワード管理      │                          │  - 通知設定           │
│                       │                          │                      │
│  Tech: Node.js/Go     │                          │  Tech: Node.js       │
│  DB: PostgreSQL       │                          │  DB: PostgreSQL      │
└───────────────────────┘                          └──────────────────────┘
            │
            │
            ├─────────────────────────────────────────────────────┐
            │                                                     │
            ▼                                                     ▼
┌───────────────────────┐                          ┌──────────────────────┐
│  家計サービス           │                          │  トランザクション      │
│  (Household Service)   │◄────────────────────────►│  サービス             │
│                       │                          │  (Transaction Svc)   │
│  - 家計管理            │                          │                      │
│  - メンバー管理        │                          │  - 取引記録管理        │
│  - 権限管理            │                          │  - CSV インポート     │
│  - 招待機能            │                          │  - 検索・フィルタ      │
│                       │                          │  - バルク操作         │
│  Tech: Node.js        │                          │                      │
│  DB: PostgreSQL       │                          │  Tech: Node.js/Go    │
└───────────────────────┘                          │  DB: PostgreSQL      │
            │                                      │  Cache: Redis        │
            │                                      └──────────────────────┘
            │                                                 │
            ▼                                                 ▼
┌───────────────────────┐                          ┌──────────────────────┐
│  カテゴリサービス       │                          │  アカウントサービス    │
│  (Category Service)    │                          │  (Account Service)   │
│                       │                          │                      │
│  - カテゴリ管理        │                          │  - 口座管理           │
│  - 階層構造管理        │                          │  - 残高管理           │
│  - デフォルト提供      │                          │  - 振替処理           │
│  - カスタマイズ        │                          │  - 明細照会           │
│                       │                          │                      │
│  Tech: Node.js        │                          │  Tech: Node.js       │
│  DB: PostgreSQL       │                          │  DB: PostgreSQL      │
└───────────────────────┘                          └──────────────────────┘
            │
            │
            ├─────────────────────────────────────────────────────┐
            │                                                     │
            ▼                                                     ▼
┌───────────────────────┐                          ┌──────────────────────┐
│  予算サービス           │                          │  分析サービス         │
│  (Budget Service)      │                          │  (Analytics Service) │
│                       │                          │                      │
│  - 予算設定            │                          │  - 月次集計           │
│  - 予算vs実績          │                          │  - カテゴリ別集計     │
│  - アラート            │                          │  - サンキー図生成     │
│  - レポート生成        │                          │  - トレンド分析       │
│                       │                          │  - 予測               │
│  Tech: Node.js        │                          │                      │
│  DB: PostgreSQL       │                          │  Tech: Python/Node.js│
└───────────────────────┘                          │  DB: PostgreSQL      │
            │                                      │  Cache: Redis        │
            │                                      └──────────────────────┘
            ▼
┌───────────────────────┐                          ┌──────────────────────┐
│  通知サービス           │                          │  ファイルサービス      │
│  (Notification Svc)    │                          │  (File Service)      │
│                       │                          │                      │
│  - メール送信          │                          │  - レシート保存       │
│  - プッシュ通知        │                          │  - 画像処理           │
│  - SMS送信            │                          │  - OCR処理            │
│  - 通知履歴            │                          │  - CSV生成           │
│                       │                          │                      │
│  Tech: Node.js        │                          │  Tech: Node.js/Go    │
│  Queue: RabbitMQ/SQS  │                          │  Storage: S3         │
└───────────────────────┘                          └──────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      共通インフラストラクチャ                       │
├─────────────────────────────────────────────────────────────────┤
│  - メッセージキュー: RabbitMQ / AWS SQS / Kafka                    │
│  - イベントストリーム: Kafka / AWS EventBridge                      │
│  - サービスメッシュ: Istio / Linkerd                               │
│  - サービスディスカバリ: Consul / AWS Cloud Map                     │
│  - 分散トレーシング: Jaeger / AWS X-Ray                            │
│  - ログ集約: ELK Stack / AWS CloudWatch                           │
│  - メトリクス: Prometheus + Grafana                               │
│  - CI/CD: GitHub Actions / GitLab CI                             │
│  - コンテナオーケストレーション: Kubernetes / AWS ECS              │
└─────────────────────────────────────────────────────────────────┘
```

---

## サービス分割の原則

### 1. ドメイン駆動設計（DDD）による分割

各サービスは明確なビジネスドメインを持つ：

- **認証ドメイン**: ユーザー認証・認可
- **ユーザードメイン**: ユーザープロフィール・設定
- **家計ドメイン**: 家計の管理・メンバー管理
- **トランザクションドメイン**: 取引の記録・管理
- **カテゴリドメイン**: カテゴリの定義・管理
- **アカウントドメイン**: 口座・カードの管理
- **予算ドメイン**: 予算設定・追跡
- **分析ドメイン**: データ集計・分析・可視化
- **通知ドメイン**: 各種通知の送信
- **ファイルドメイン**: ファイル管理・処理

### 2. 単一責任の原則

各サービスは1つの責任のみを持つ。

### 3. 疎結合・高凝集

サービス間は疎結合、サービス内は高凝集。

### 4. データベース分離

各サービスは独自のデータベースを持つ（Database per Service パターン）。

---

## 各サービスの詳細

### 1. 認証サービス (Auth Service)

#### 責務
- ユーザー登録・ログイン・ログアウト
- JWT トークンの発行・検証
- パスワード管理（変更・リセット）
- セッション管理

#### API エンドポイント
```
POST   /api/auth/signup         - ユーザー登録
POST   /api/auth/login          - ログイン
POST   /api/auth/logout         - ログアウト
POST   /api/auth/refresh        - トークンリフレッシュ
POST   /api/auth/verify         - トークン検証（内部API）
POST   /api/auth/password/change    - パスワード変更
POST   /api/auth/password/reset     - パスワードリセット
```

#### データモデル
```prisma
model AuthUser {
  id            String   @id @default(uuid())
  email         String   @unique
  passwordHash  String
  isVerified    Boolean  @default(false)
  lastLoginAt   DateTime?
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
}

model RefreshToken {
  id        String   @id @default(uuid())
  userId    String
  token     String   @unique
  expiresAt DateTime
  createdAt DateTime @default(now())
}
```

#### 技術スタック
- **言語**: Node.js (Fastify) または Go
- **DB**: PostgreSQL
- **キャッシュ**: Redis (ブラックリストトークン)
- **暗号化**: bcrypt, JWT

---

### 2. ユーザーサービス (User Service)

#### 責務
- ユーザープロフィール管理
- ユーザー設定管理
- 通知設定
- アバター管理

#### API エンドポイント
```
GET    /api/users/me            - 自分の情報取得
PUT    /api/users/me            - プロフィール更新
GET    /api/users/settings      - 設定取得
PUT    /api/users/settings      - 設定更新
POST   /api/users/avatar        - アバターアップロード
```

#### データモデル
```prisma
model User {
  id          String   @id @default(uuid())
  authId      String   @unique // 認証サービスのユーザーID
  email       String   @unique
  displayName String?
  avatarUrl   String?
  timezone    String   @default("Asia/Tokyo")
  locale      String   @default("ja")
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

model UserSettings {
  id                String   @id @default(uuid())
  userId            String   @unique
  fiscalYearStart   Int      @default(1)
  currency          String   @default("JPY")
  notifyEmail       Boolean  @default(true)
  notifyPush        Boolean  @default(true)
  budgetAlert       Boolean  @default(true)
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
}
```

#### 技術スタック
- **言語**: Node.js (Fastify)
- **DB**: PostgreSQL

---

### 3. 家計サービス (Household Service)

#### 責務
- 家計の作成・管理・削除
- 家族メンバーの管理
- 招待・権限管理
- アクセス制御

#### API エンドポイント
```
POST   /api/households                    - 家計作成
GET    /api/households                    - 家計一覧
GET    /api/households/:id                - 家計詳細
PUT    /api/households/:id                - 家計更新
DELETE /api/households/:id                - 家計削除

GET    /api/households/:id/members        - メンバー一覧
POST   /api/households/:id/invite         - メンバー招待
DELETE /api/households/:id/members/:uid   - メンバー削除
PUT    /api/households/:id/members/:uid   - 権限変更
```

#### データモデル
```prisma
model Household {
  id              String   @id @default(uuid())
  name            String
  description     String?
  ownerId         String
  fiscalYearStart Int      @default(1)
  currency        String   @default("JPY")
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
}

model HouseholdMember {
  id          String     @id @default(uuid())
  householdId String
  userId      String
  role        MemberRole @default(viewer)
  joinedAt    DateTime   @default(now())
  
  @@unique([householdId, userId])
}

enum MemberRole {
  owner
  editor
  viewer
}

model Invitation {
  id          String   @id @default(uuid())
  householdId String
  email       String
  role        MemberRole
  token       String   @unique
  expiresAt   DateTime
  createdAt   DateTime @default(now())
}
```

#### 技術スタック
- **言語**: Node.js (Fastify)
- **DB**: PostgreSQL
- **イベント**: RabbitMQ (メンバー追加イベント)

---

### 4. トランザクションサービス (Transaction Service)

#### 責務
- 取引の記録・更新・削除
- CSV インポート・エクスポート
- 検索・フィルタリング
- ページネーション

#### API エンドポイント
```
POST   /api/transactions                  - 取引作成
GET    /api/transactions                  - 取引一覧
GET    /api/transactions/:id              - 取引詳細
PUT    /api/transactions/:id              - 取引更新
DELETE /api/transactions/:id              - 取引削除
POST   /api/transactions/bulk             - 一括作成
POST   /api/transactions/import/csv       - CSVインポート
GET    /api/transactions/export/csv       - CSVエクスポート
```

#### データモデル
```prisma
model Transaction {
  id              String          @id @default(uuid())
  householdId     String
  accountId       String?
  categoryId      String?
  date            DateTime
  amount          Decimal         @db.Decimal(15, 2)
  type            TransactionType
  description     String?
  memo            String?
  receiptUrl      String?
  isRecurring     Boolean         @default(false)
  recurringGroupId String?
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt
  
  @@index([householdId, date])
  @@index([householdId, categoryId])
}

enum TransactionType {
  income
  expense
  transfer
}

model TransactionTag {
  transactionId String
  tagId         String
  
  @@id([transactionId, tagId])
}
```

#### 技術スタック
- **言語**: Node.js (Fastify) または Go（高パフォーマンス）
- **DB**: PostgreSQL
- **キャッシュ**: Redis (集計結果)
- **イベント**: Kafka (トランザクション作成イベント)

---

### 5. カテゴリサービス (Category Service)

#### 責務
- カテゴリの作成・管理・削除
- 階層構造の管理
- デフォルトカテゴリの提供
- カテゴリのカスタマイズ

#### API エンドポイント
```
GET    /api/categories                    - カテゴリ一覧
POST   /api/categories                    - カテゴリ作成
GET    /api/categories/:id                - カテゴリ詳細
PUT    /api/categories/:id                - カテゴリ更新
DELETE /api/categories/:id                - カテゴリ削除
GET    /api/categories/defaults           - デフォルトカテゴリ取得
POST   /api/categories/import/defaults    - デフォルトをインポート
```

#### データモデル
```prisma
model Category {
  id           String          @id @default(uuid())
  householdId  String?         // null = システムデフォルト
  name         String
  type         TransactionType
  parentId     String?
  color        String?
  icon         String?
  isDefault    Boolean         @default(false)
  isActive     Boolean         @default(true)
  displayOrder Int             @default(0)
  createdAt    DateTime        @default(now())
  updatedAt    DateTime        @updatedAt
  
  @@index([householdId, type])
}
```

#### 技術スタック
- **言語**: Node.js (Fastify)
- **DB**: PostgreSQL
- **キャッシュ**: Redis (デフォルトカテゴリ)

---

### 6. アカウントサービス (Account Service)

#### 責務
- 口座・カードの管理
- 残高管理
- 振替処理
- 口座明細の照会

#### API エンドポイント
```
GET    /api/accounts                      - アカウント一覧
POST   /api/accounts                      - アカウント作成
GET    /api/accounts/:id                  - アカウント詳細
PUT    /api/accounts/:id                  - アカウント更新
DELETE /api/accounts/:id                  - アカウント削除
GET    /api/accounts/:id/balance          - 残高取得
POST   /api/accounts/transfer             - 振替処理
```

#### データモデル
```prisma
model Account {
  id             String      @id @default(uuid())
  householdId    String
  name           String
  type           AccountType
  initialBalance Decimal     @db.Decimal(15, 2) @default(0)
  currentBalance Decimal     @db.Decimal(15, 2) @default(0)
  isActive       Boolean     @default(true)
  displayOrder   Int         @default(0)
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt
  
  @@index([householdId])
}

enum AccountType {
  cash
  bank
  credit_card
  e_money
  investment
  other
}

model BalanceSnapshot {
  id          String   @id @default(uuid())
  accountId   String
  date        DateTime
  balance     Decimal  @db.Decimal(15, 2)
  createdAt   DateTime @default(now())
  
  @@index([accountId, date])
}
```

#### 技術スタック
- **言語**: Node.js (Fastify)
- **DB**: PostgreSQL
- **イベント**: Kafka (残高変更イベント)

---

### 7. 予算サービス (Budget Service)

#### 責務
- 予算の設定・管理
- 予算vs実績の計算
- 予算超過アラート
- レポート生成

#### API エンドポイント
```
GET    /api/budgets                       - 予算一覧
POST   /api/budgets                       - 予算設定
GET    /api/budgets/:id                   - 予算詳細
PUT    /api/budgets/:id                   - 予算更新
DELETE /api/budgets/:id                   - 予算削除
GET    /api/budgets/vs-actual             - 予算vs実績
GET    /api/budgets/reports/monthly       - 月次レポート
```

#### データモデル
```prisma
model Budget {
  id          String   @id @default(uuid())
  householdId String
  categoryId  String
  year        Int
  month       Int
  amount      Decimal  @db.Decimal(15, 2)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@unique([householdId, categoryId, year, month])
  @@index([householdId, year, month])
}

model BudgetAlert {
  id          String   @id @default(uuid())
  budgetId    String
  threshold   Int      // パーセント（例: 80, 100）
  isTriggered Boolean  @default(false)
  triggeredAt DateTime?
  createdAt   DateTime @default(now())
}
```

#### 技術スタック
- **言語**: Node.js (Fastify)
- **DB**: PostgreSQL
- **イベント**: RabbitMQ (予算アラートイベント)

---

### 8. 分析サービス (Analytics Service)

#### 責務
- 月次・年次集計
- カテゴリ別集計
- サンキー図データ生成
- トレンド分析
- 予測

#### API エンドポイント
```
GET    /api/analytics/monthly             - 月次集計
GET    /api/analytics/category            - カテゴリ別集計
GET    /api/analytics/sankey              - サンキー図データ
GET    /api/analytics/trends              - トレンド分析
GET    /api/analytics/forecast            - 予測データ
GET    /api/analytics/dashboard           - ダッシュボードデータ
```

#### データモデル
```prisma
// 集計結果をキャッシュ
model AggregationCache {
  id          String   @id @default(uuid())
  householdId String
  type        String   // monthly, category, etc.
  year        Int
  month       Int?
  data        Json
  expiresAt   DateTime
  createdAt   DateTime @default(now())
  
  @@index([householdId, type, year, month])
}
```

#### 技術スタック
- **言語**: Python (FastAPI) または Node.js
- **DB**: PostgreSQL (読み取り専用レプリカ)
- **キャッシュ**: Redis (集計結果)
- **イベント**: Kafka (トランザクション作成イベントを購読)
- **分析**: pandas, NumPy (Python の場合)

---

### 9. 通知サービス (Notification Service)

#### 責務
- メール送信
- プッシュ通知
- SMS送信
- 通知履歴の管理

#### API エンドポイント
```
POST   /api/notifications/email           - メール送信
POST   /api/notifications/push            - プッシュ通知
POST   /api/notifications/sms             - SMS送信
GET    /api/notifications/history         - 通知履歴
```

#### データモデル
```prisma
model NotificationLog {
  id          String           @id @default(uuid())
  userId      String
  type        NotificationType
  channel     String           // email, push, sms
  subject     String?
  message     String
  status      String           // sent, failed, pending
  sentAt      DateTime?
  createdAt   DateTime         @default(now())
  
  @@index([userId, createdAt])
}

enum NotificationType {
  budget_alert
  monthly_report
  invitation
  transaction_reminder
  system
}
```

#### 技術スタック
- **言語**: Node.js (Fastify)
- **DB**: PostgreSQL
- **キュー**: RabbitMQ または AWS SQS
- **メール**: SendGrid, AWS SES
- **プッシュ**: Firebase Cloud Messaging (FCM)

---

### 10. ファイルサービス (File Service)

#### 責務
- レシート画像の保存
- 画像処理（圧縮、リサイズ）
- OCR処理
- CSV生成・エクスポート

#### API エンドポイント
```
POST   /api/files/upload                  - ファイルアップロード
GET    /api/files/:id                     - ファイル取得
DELETE /api/files/:id                     - ファイル削除
POST   /api/files/ocr                     - OCR処理
POST   /api/files/export/csv              - CSVエクスポート
```

#### データモデル
```prisma
model File {
  id          String   @id @default(uuid())
  householdId String
  userId      String
  type        String   // receipt, avatar, etc.
  filename    String
  mimeType    String
  size        Int
  url         String
  thumbnailUrl String?
  createdAt   DateTime @default(now())
  
  @@index([householdId, type])
}

model OCRResult {
  id          String   @id @default(uuid())
  fileId      String   @unique
  text        String
  amount      Decimal? @db.Decimal(15, 2)
  date        DateTime?
  merchant    String?
  confidence  Float
  createdAt   DateTime @default(now())
}
```

#### 技術スタック
- **言語**: Node.js (Fastify) または Go
- **ストレージ**: AWS S3 / Google Cloud Storage
- **OCR**: Google Cloud Vision API / AWS Textract
- **画像処理**: Sharp (Node.js) / ImageMagick

---

## サービス間通信

### 1. 同期通信（REST API）

サービス間の直接呼び出しが必要な場合：

```typescript
// 例: トランザクションサービス → カテゴリサービス
const category = await categoryServiceClient.getCategory(categoryId);
```

**使用ケース**:
- リアルタイムで結果が必要な場合
- データの一貫性が重要な場合

### 2. 同期通信（gRPC）

高速・低レイテンシのサービス間通信：

#### gRPC サービス定義

```protobuf
// proto/category/v1/category.proto
syntax = "proto3";

package category.v1;

import "google/protobuf/timestamp.proto";

service CategoryService {
  rpc GetCategory(GetCategoryRequest) returns (GetCategoryResponse);
  rpc ListCategories(ListCategoriesRequest) returns (ListCategoriesResponse);
  rpc CreateCategory(CreateCategoryRequest) returns (CreateCategoryResponse);
  rpc UpdateCategory(UpdateCategoryRequest) returns (UpdateCategoryResponse);
  rpc DeleteCategory(DeleteCategoryRequest) returns (DeleteCategoryResponse);
}

message Category {
  string id = 1;
  string household_id = 2;
  string name = 3;
  TransactionType type = 4;
  optional string parent_id = 5;
  optional string color = 6;
  optional string icon = 7;
  bool is_default = 8;
  bool is_active = 9;
  int32 display_order = 10;
  google.protobuf.Timestamp created_at = 11;
  google.protobuf.Timestamp updated_at = 12;
}

enum TransactionType {
  TRANSACTION_TYPE_UNSPECIFIED = 0;
  TRANSACTION_TYPE_INCOME = 1;
  TRANSACTION_TYPE_EXPENSE = 2;
  TRANSACTION_TYPE_TRANSFER = 3;
}

message GetCategoryRequest {
  string id = 1;
  string household_id = 2;
}

message GetCategoryResponse {
  Category category = 1;
}

message ListCategoriesRequest {
  string household_id = 1;
  optional TransactionType type = 2;
  optional string parent_id = 3;
  bool include_inactive = 4;
  int32 page_size = 5;
  string page_token = 6;
}

message ListCategoriesResponse {
  repeated Category categories = 1;
  string next_page_token = 2;
  int32 total_count = 3;
}

message CreateCategoryRequest {
  string household_id = 1;
  string name = 2;
  TransactionType type = 3;
  optional string parent_id = 4;
  optional string color = 5;
  optional string icon = 6;
  int32 display_order = 7;
}

message CreateCategoryResponse {
  Category category = 1;
}

message UpdateCategoryRequest {
  string id = 1;
  string household_id = 2;
  optional string name = 3;
  optional string color = 4;
  optional string icon = 5;
  optional bool is_active = 6;
  optional int32 display_order = 7;
}

message UpdateCategoryResponse {
  Category category = 1;
}

message DeleteCategoryRequest {
  string id = 1;
  string household_id = 2;
}

message DeleteCategoryResponse {
  bool success = 1;
}
```

```protobuf
// proto/transaction/v1/transaction.proto
syntax = "proto3";

package transaction.v1;

import "google/protobuf/timestamp.proto";
import "category/v1/category.proto";

service TransactionService {
  rpc CreateTransaction(CreateTransactionRequest) returns (CreateTransactionResponse);
  rpc GetTransaction(GetTransactionRequest) returns (GetTransactionResponse);
  rpc ListTransactions(ListTransactionsRequest) returns (ListTransactionsResponse);
  rpc UpdateTransaction(UpdateTransactionRequest) returns (UpdateTransactionResponse);
  rpc DeleteTransaction(DeleteTransactionRequest) returns (DeleteTransactionResponse);
  rpc BatchCreateTransactions(BatchCreateTransactionsRequest) returns (BatchCreateTransactionsResponse);
}

message Transaction {
  string id = 1;
  string household_id = 2;
  optional string account_id = 3;
  optional string category_id = 4;
  google.protobuf.Timestamp date = 5;
  string amount = 6;  // Decimal as string
  category.v1.TransactionType type = 7;
  optional string description = 8;
  optional string memo = 9;
  optional string receipt_url = 10;
  bool is_recurring = 11;
  optional string recurring_group_id = 12;
  google.protobuf.Timestamp created_at = 13;
  google.protobuf.Timestamp updated_at = 14;
}

message CreateTransactionRequest {
  string household_id = 1;
  optional string account_id = 2;
  optional string category_id = 3;
  google.protobuf.Timestamp date = 4;
  string amount = 5;
  category.v1.TransactionType type = 6;
  optional string description = 7;
  optional string memo = 8;
  optional string receipt_url = 9;
  bool is_recurring = 10;
  optional string recurring_group_id = 11;
}

message CreateTransactionResponse {
  Transaction transaction = 1;
}

message GetTransactionRequest {
  string id = 1;
  string household_id = 2;
}

message GetTransactionResponse {
  Transaction transaction = 1;
}

message ListTransactionsRequest {
  string household_id = 1;
  optional google.protobuf.Timestamp start_date = 2;
  optional google.protobuf.Timestamp end_date = 3;
  optional string account_id = 4;
  optional string category_id = 5;
  optional category.v1.TransactionType type = 6;
  int32 page_size = 7;
  string page_token = 8;
}

message ListTransactionsResponse {
  repeated Transaction transactions = 1;
  string next_page_token = 2;
  int32 total_count = 3;
}

message UpdateTransactionRequest {
  string id = 1;
  string household_id = 2;
  optional string account_id = 3;
  optional string category_id = 4;
  optional google.protobuf.Timestamp date = 5;
  optional string amount = 6;
  optional category.v1.TransactionType type = 7;
  optional string description = 8;
  optional string memo = 9;
  optional string receipt_url = 10;
}

message UpdateTransactionResponse {
  Transaction transaction = 1;
}

message DeleteTransactionRequest {
  string id = 1;
  string household_id = 2;
}

message DeleteTransactionResponse {
  bool success = 1;
}

message BatchCreateTransactionsRequest {
  string household_id = 1;
  repeated CreateTransactionRequest transactions = 2;
}

message BatchCreateTransactionsResponse {
  repeated Transaction transactions = 1;
  int32 success_count = 2;
  int32 failure_count = 3;
  repeated string error_messages = 4;
}
```

```protobuf
// proto/analytics/v1/analytics.proto
syntax = "proto3";

package analytics.v1;

import "google/protobuf/timestamp.proto";

service AnalyticsService {
  rpc GetMonthlySummary(GetMonthlySummaryRequest) returns (GetMonthlySummaryResponse);
  rpc GetCategorySummary(GetCategorySummaryRequest) returns (GetCategorySummaryResponse);
  rpc GetSankeyData(GetSankeyDataRequest) returns (GetSankeyDataResponse);
  rpc GetTrendAnalysis(GetTrendAnalysisRequest) returns (GetTrendAnalysisResponse);
}

message GetMonthlySummaryRequest {
  string household_id = 1;
  int32 year = 2;
  int32 month = 3;
}

message GetMonthlySummaryResponse {
  string household_id = 1;
  int32 year = 2;
  int32 month = 3;
  string total_income = 4;
  string total_expense = 5;
  string net_amount = 6;
  repeated CategoryAmount category_breakdown = 7;
  google.protobuf.Timestamp calculated_at = 8;
}

message CategoryAmount {
  string category_id = 1;
  string category_name = 2;
  string amount = 3;
  double percentage = 4;
}

message GetCategorySummaryRequest {
  string household_id = 1;
  google.protobuf.Timestamp start_date = 2;
  google.protobuf.Timestamp end_date = 3;
}

message GetCategorySummaryResponse {
  repeated CategoryAmount categories = 1;
  google.protobuf.Timestamp calculated_at = 2;
}

message GetSankeyDataRequest {
  string household_id = 1;
  int32 year = 2;
  int32 month = 3;
}

message GetSankeyDataResponse {
  repeated SankeyNode nodes = 1;
  repeated SankeyLink links = 2;
  google.protobuf.Timestamp calculated_at = 3;
}

message SankeyNode {
  string id = 1;
  string name = 2;
}

message SankeyLink {
  string source = 1;
  string target = 2;
  string value = 3;
}

message GetTrendAnalysisRequest {
  string household_id = 1;
  google.protobuf.Timestamp start_date = 2;
  google.protobuf.Timestamp end_date = 3;
  string granularity = 4;  // daily, weekly, monthly
}

message GetTrendAnalysisResponse {
  repeated DataPoint income_trend = 1;
  repeated DataPoint expense_trend = 2;
  repeated DataPoint net_trend = 3;
  google.protobuf.Timestamp calculated_at = 4;
}

message DataPoint {
  google.protobuf.Timestamp date = 1;
  string value = 2;
}
```

#### gRPC クライアント実装例（Node.js）

```typescript
// services/transaction/src/clients/categoryClient.ts
import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';
import path from 'path';

const PROTO_PATH = path.join(__dirname, '../../../../proto/category/v1/category.proto');

const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
});

const categoryProto = grpc.loadPackageDefinition(packageDefinition).category.v1 as any;

export class CategoryServiceClient {
  private client: any;

  constructor(serviceUrl: string) {
    this.client = new categoryProto.CategoryService(
      serviceUrl,
      grpc.credentials.createInsecure() // 本番環境では createSsl() を使用
    );
  }

  async getCategory(id: string, householdId: string): Promise<any> {
    return new Promise((resolve, reject) => {
      this.client.GetCategory(
        { id, household_id: householdId },
        (error: grpc.ServiceError | null, response: any) => {
          if (error) {
            reject(error);
          } else {
            resolve(response.category);
          }
        }
      );
    });
  }

  async listCategories(
    householdId: string,
    type?: string,
    pageSize: number = 100
  ): Promise<any> {
    return new Promise((resolve, reject) => {
      this.client.ListCategories(
        {
          household_id: householdId,
          type,
          page_size: pageSize,
        },
        (error: grpc.ServiceError | null, response: any) => {
          if (error) {
            reject(error);
          } else {
            resolve({
              categories: response.categories,
              nextPageToken: response.next_page_token,
              totalCount: response.total_count,
            });
          }
        }
      );
    });
  }

  async createCategory(data: any): Promise<any> {
    return new Promise((resolve, reject) => {
      this.client.CreateCategory(data, (error: grpc.ServiceError | null, response: any) => {
        if (error) {
          reject(error);
        } else {
          resolve(response.category);
        }
      });
    });
  }

  close() {
    grpc.closeClient(this.client);
  }
}

// 使用例
const categoryClient = new CategoryServiceClient('category-service:50051');

try {
  const category = await categoryClient.getCategory(categoryId, householdId);
  console.log('Category:', category);
} catch (error) {
  console.error('gRPC Error:', error);
}
```

#### gRPC サーバー実装例（Node.js）

```typescript
// services/category/src/grpc/server.ts
import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';
import path from 'path';
import { CategoryService } from '../services/categoryService';

const PROTO_PATH = path.join(__dirname, '../../../../proto/category/v1/category.proto');

const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
});

const categoryProto = grpc.loadPackageDefinition(packageDefinition).category.v1 as any;

const categoryService = new CategoryService();

// gRPC サービス実装
const grpcServer = new grpc.Server();

grpcServer.addService(categoryProto.CategoryService.service, {
  GetCategory: async (call: any, callback: any) => {
    try {
      const { id, household_id } = call.request;
      const category = await categoryService.getById(id, household_id);
      
      callback(null, { category });
    } catch (error: any) {
      callback({
        code: grpc.status.INTERNAL,
        message: error.message,
      });
    }
  },

  ListCategories: async (call: any, callback: any) => {
    try {
      const {
        household_id,
        type,
        parent_id,
        include_inactive,
        page_size,
        page_token,
      } = call.request;

      const result = await categoryService.list({
        householdId: household_id,
        type,
        parentId: parent_id,
        includeInactive: include_inactive,
        pageSize: page_size,
        pageToken: page_token,
      });

      callback(null, {
        categories: result.categories,
        next_page_token: result.nextPageToken,
        total_count: result.totalCount,
      });
    } catch (error: any) {
      callback({
        code: grpc.status.INTERNAL,
        message: error.message,
      });
    }
  },

  CreateCategory: async (call: any, callback: any) => {
    try {
      const data = call.request;
      const category = await categoryService.create({
        householdId: data.household_id,
        name: data.name,
        type: data.type,
        parentId: data.parent_id,
        color: data.color,
        icon: data.icon,
        displayOrder: data.display_order,
      });

      callback(null, { category });
    } catch (error: any) {
      callback({
        code: grpc.status.INTERNAL,
        message: error.message,
      });
    }
  },

  UpdateCategory: async (call: any, callback: any) => {
    try {
      const data = call.request;
      const category = await categoryService.update(data.id, {
        householdId: data.household_id,
        name: data.name,
        color: data.color,
        icon: data.icon,
        isActive: data.is_active,
        displayOrder: data.display_order,
      });

      callback(null, { category });
    } catch (error: any) {
      callback({
        code: grpc.status.INTERNAL,
        message: error.message,
      });
    }
  },

  DeleteCategory: async (call: any, callback: any) => {
    try {
      const { id, household_id } = call.request;
      await categoryService.delete(id, household_id);

      callback(null, { success: true });
    } catch (error: any) {
      callback({
        code: grpc.status.INTERNAL,
        message: error.message,
      });
    }
  },
});

// サーバー起動
const PORT = process.env.GRPC_PORT || '50051';
grpcServer.bindAsync(
  `0.0.0.0:${PORT}`,
  grpc.ServerCredentials.createInsecure(), // 本番環境では createSsl() を使用
  (error, port) => {
    if (error) {
      console.error('Failed to start gRPC server:', error);
      process.exit(1);
    }
    console.log(`gRPC server running on port ${port}`);
    grpcServer.start();
  }
);
```

#### gRPC とREST APIの併用

```typescript
// services/transaction/src/index.ts
import fastify from 'fastify';
import { startGrpcServer } from './grpc/server';
import { transactionRoutes } from './http/routes';

// REST API サーバー（外部クライアント向け）
const httpServer = fastify();
httpServer.register(transactionRoutes, { prefix: '/api/transactions' });

const HTTP_PORT = process.env.HTTP_PORT || 3000;
httpServer.listen({ port: HTTP_PORT, host: '0.0.0.0' }, (err) => {
  if (err) {
    console.error(err);
    process.exit(1);
  }
  console.log(`HTTP server listening on port ${HTTP_PORT}`);
});

// gRPC サーバー（サービス間通信用）
startGrpcServer();
```

#### Kubernetes ServiceMesh での gRPC サポート

```yaml
# k8s/category-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: category-service
  labels:
    app: category-service
spec:
  ports:
  - name: http
    port: 80
    targetPort: 3000
    protocol: TCP
  - name: grpc
    port: 50051
    targetPort: 50051
    protocol: TCP
    appProtocol: grpc  # gRPC プロトコルを明示
  selector:
    app: category-service
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: category-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: category-service
  template:
    metadata:
      labels:
        app: category-service
    spec:
      containers:
      - name: category-service
        image: category-service:v1.0.0
        ports:
        - name: http
          containerPort: 3000
          protocol: TCP
        - name: grpc
          containerPort: 50051
          protocol: TCP
        env:
        - name: HTTP_PORT
          value: "3000"
        - name: GRPC_PORT
          value: "50051"
```

**gRPC の使用ケース**:
- サービス間の高頻度な同期通信
- 低レイテンシが求められる処理
- 型安全な通信が必要な場合
- ストリーミング（サーバーストリーム、双方向ストリーム）
- 内部マイクロサービス間通信

**gRPC のメリット**:
- REST APIより高速（Protocol Buffers使用）
- 型安全（.protoファイルから自動生成）
- ストリーミングサポート
- 多言語サポート（コード生成）
- HTTP/2による多重化

**gRPC のデメリット**:
- ブラウザからの直接呼び出しが困難（gRPC-Web必要）
- デバッグがRESTより難しい
- .protoファイルの管理が必要

### 3. 非同期通信（メッセージキュー）

イベント駆動アーキテクチャ：

```typescript
// トランザクション作成時
eventBus.publish('transaction.created', {
  transactionId: '...',
  householdId: '...',
  amount: 50000,
  categoryId: '...',
  date: '2024-08-15'
});

// 分析サービスが購読
eventBus.subscribe('transaction.created', async (event) => {
  await updateAggregations(event);
});

// 予算サービスが購読
eventBus.subscribe('transaction.created', async (event) => {
  await checkBudgetAlert(event);
});
```

**使用ケース**:
- リアルタイム性が不要な処理
- 複数のサービスに同じイベントを通知
- 処理の分散化

### 4. イベントソーシング（オプション）

すべての状態変更をイベントとして記録：

```typescript
// イベントストア
const events = [
  { type: 'TransactionCreated', data: {...} },
  { type: 'TransactionUpdated', data: {...} },
  { type: 'TransactionDeleted', data: {...} }
];

// イベントから現在の状態を再構築
const currentState = events.reduce((state, event) => {
  return applyEvent(state, event);
}, initialState);
```

---

## データ整合性の戦略

### 1. Saga パターン

分散トランザクションを複数のローカルトランザクションに分割：

```typescript
// 家計作成 Saga
async function createHouseholdSaga(data) {
  // Step 1: 家計サービス - 家計作成
  const household = await householdService.create(data);
  
  try {
    // Step 2: カテゴリサービス - デフォルトカテゴリ追加
    await categoryService.importDefaults(household.id);
    
    // Step 3: アカウントサービス - デフォルトアカウント作成
    await accountService.createDefault(household.id);
    
    return household;
  } catch (error) {
    // 補償トランザクション（ロールバック）
    await householdService.delete(household.id);
    throw error;
  }
}
```

### 2. 最終的整合性

リアルタイムでの整合性は保証せず、最終的に整合性を確保：

```typescript
// 例: トランザクション作成後、分析データは非同期で更新
// ユーザーには「処理中」と表示し、数秒後に反映
```

### 3. CQRS (Command Query Responsibility Segregation)

読み取りと書き込みを分離：

```typescript
// 書き込み側（Command）
class TransactionCommandService {
  async create(data) { ... }
  async update(id, data) { ... }
  async delete(id) { ... }
}

// 読み取り側（Query）
class TransactionQueryService {
  async getById(id) { ... }
  async getList(filters) { ... }
  async getAggregations() { ... }
}
```

---

## セキュリティ

### 1. API Gateway レベル

- JWT トークンの検証
- レート制限
- IP ホワイトリスト
- CORS 設定

### 2. サービスレベル

- サービス間認証（mTLS）
- 権限チェック（RBAC）
- データアクセス制御（household単位）
- 入力バリデーション

### 3. データベースレベル

- Row Level Security (RLS)
- データ暗号化
- バックアップ・リストア

---

## スケーラビリティ

### 1. 水平スケーリング

各サービスを独立してスケール：

```yaml
# Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: transaction-service
spec:
  replicas: 5  # 負荷に応じて増減
  template:
    spec:
      containers:
      - name: transaction-service
        image: transaction-service:latest
```

### 2. データベース分離

各サービスが独自のデータベースを持つことで、データベースもスケール可能：

- トランザクションサービス: 読み取りレプリカ使用
- 分析サービス: 専用の読み取り専用DB

### 3. キャッシュ戦略

- Redis によるデータキャッシュ
- CDN による静的ファイルキャッシュ
- API Gateway レベルのキャッシュ

---

## 監視・ログ

### 1. 分散トレーシング

リクエストの流れを追跡：

```
User Request → API Gateway → Auth Service → Household Service
                                          → Transaction Service
                                          → Analytics Service
```

**ツール**: Jaeger, Zipkin, AWS X-Ray

### 2. ログ集約

各サービスのログを一元管理：

```
Transaction Service → Fluentd → Elasticsearch → Kibana
Budget Service      →
Analytics Service   →
```

**ツール**: ELK Stack, AWS CloudWatch Logs

### 3. メトリクス収集

パフォーマンスメトリクスの収集：

```
Prometheus → Grafana
```

**メトリクス**:
- リクエスト数
- レスポンスタイム
- エラー率
- CPU・メモリ使用率

---

## デプロイ戦略

### 1. コンテナ化

各サービスをDockerコンテナ化：

```dockerfile
# transaction-service/Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### 2. Kubernetes デプロイ

```yaml
apiVersion: v1
kind: Service
metadata:
  name: transaction-service
spec:
  selector:
    app: transaction-service
  ports:
  - port: 80
    targetPort: 3000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: transaction-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: transaction-service
  template:
    metadata:
      labels:
        app: transaction-service
    spec:
      containers:
      - name: transaction-service
        image: transaction-service:v1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secrets
              key: url
```

### 3. CI/CD パイプライン

```yaml
# .github/workflows/transaction-service.yml
name: Transaction Service CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'services/transaction/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          cd services/transaction
          npm test
  
  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker image
        run: docker build -t transaction-service .
      
      - name: Push to registry
        run: docker push transaction-service:latest
      
      - name: Deploy to Kubernetes
        run: kubectl apply -f k8s/
```

---

## マイグレーション計画

### Phase 1: 基盤構築（2-3週間）

1. **インフラ構築**
   - Kubernetes クラスタ構築
   - API Gateway 構築
   - メッセージキュー構築
   - 監視システム構築

2. **認証サービス実装**
   - JWT 認証実装
   - ユーザー管理機能

### Phase 2: コアサービス実装（4-6週間）

1. **家計サービス**
2. **トランザクションサービス**
3. **カテゴリサービス**
4. **アカウントサービス**

### Phase 3: 拡張サービス実装（3-4週間）

1. **予算サービス**
2. **分析サービス**
3. **通知サービス**
4. **ファイルサービス**

### Phase 4: 統合・テスト（2-3週間）

1. E2Eテスト
2. パフォーマンステスト
3. セキュリティテスト
4. 負荷テスト

### Phase 5: 移行・リリース（2週間）

1. データ移行
2. カナリアリリース
3. モニタリング
4. フルリリース

---

## コスト見積もり

### AWS 環境（月額）

| サービス | 内容 | 月額（USD） |
|---------|------|------------|
| EKS | Kubernetesクラスタ | $144 |
| EC2 | ワーカーノード (t3.medium × 5) | $180 |
| RDS | PostgreSQL (db.t3.medium × 10) | $700 |
| ElastiCache | Redis (cache.t3.micro × 3) | $45 |
| S3 | ファイルストレージ | $50 |
| CloudFront | CDN | $30 |
| Application Load Balancer | API Gateway | $25 |
| CloudWatch | ログ・メトリクス | $100 |
| SQS / SNS | メッセージキュー | $20 |
| **合計** | | **$1,294** |

### スタートアップ向け最小構成（月額 $300-400）

- ECS Fargate (サーバーレスコンテナ)
- RDS Aurora Serverless (必要時のみ課金)
- Redis: ElastiCache の代わりに EC2 上の Redis
- S3 + CloudFront
- Application Load Balancer

---

## まとめ

### メリット

✅ **スケーラビリティ**: 各サービスを独立してスケール可能
✅ **保守性**: サービスごとに独立して開発・デプロイ可能
✅ **障害隔離**: 1つのサービスの障害が他に波及しない
✅ **技術選択の柔軟性**: サービスごとに最適な技術を選択可能
✅ **チーム分割**: サービスごとにチームを分けて並行開発可能

### デメリット・課題

❌ **複雑性の増加**: モノリスより運用が複雑
❌ **分散トランザクション**: データ整合性の管理が難しい
❌ **ネットワークレイテンシ**: サービス間通信のオーバーヘッド
❌ **デバッグの困難さ**: 分散トレーシングが必須
❌ **初期コスト**: インフラ構築に時間とコストがかかる

### 推奨事項

- **段階的な移行**: 最初はモノリスで構築し、必要に応じてマイクロサービス化
- **適切な粒度**: 過度に細かく分割しない
- **DevOps文化**: CI/CD、監視、自動化が必須
- **ドキュメント**: サービス間の契約を明確に文書化
