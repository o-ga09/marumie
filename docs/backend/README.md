# バックエンドAPIドキュメント - 目次

このディレクトリには、政治資金ダッシュボードのバックエンドAPIに関する包括的な仕様書が含まれています。

---

## ドキュメント一覧

### 📋 [00_overview.md](./00_overview.md)
バックエンドAPIの概要、アーキテクチャ、技術スタック、マイグレーション戦略について説明します。

**主な内容:**
- 現状のアーキテクチャと目指す構成
- 技術スタック（Fastify + tRPC推奨）
- API設計原則
- 主要機能カテゴリ
- マイグレーション戦略
- セキュリティ考慮事項

---

### 📊 [01_data_models.md](./01_data_models.md)
システムで使用するすべてのデータモデルを定義します。

**主な内容:**
- PoliticalOrganization（政治団体）
- Transaction（取引）
- BalanceSnapshot（残高スナップショット）
- User（ユーザー）
- DisplayTransaction（表示用トランザクション）
- SankeyData（サンキー図データ）
- BalanceSheetData（貸借対照表データ）
- MonthlyAggregation（月次集計データ）
- データベース図

---

### 🏛️ [02_political_organizations.md](./02_political_organizations.md)
政治団体管理に関するAPIエンドポイント仕様。

**エンドポイント:**
- `GET /api/political-organizations` - 一覧取得
- `GET /api/political-organizations/:slug` - 詳細取得
- `POST /api/political-organizations` - 作成（管理者専用）
- `PUT /api/political-organizations/:id` - 更新（管理者専用）
- `DELETE /api/political-organizations/:id` - 削除（管理者専用）

---

### 💰 [03_transactions.md](./03_transactions.md)
トランザクション管理に関するAPIエンドポイント仕様。

**エンドポイント:**
- `GET /api/transactions` - 一覧取得（フィルタリング、ページネーション対応）
- `GET /api/transactions/:id` - 詳細取得
- `GET /api/transactions/export/csv` - CSVエクスポート
- `POST /api/transactions/preview` - CSVプレビュー（管理者専用）
- `POST /api/transactions/upload` - 一括登録（管理者専用）
- `DELETE /api/transactions` - 全削除（管理者専用）

---

### 📈 [04_analytics.md](./04_analytics.md)
データ集計・可視化に関するAPIエンドポイント仕様。

**エンドポイント:**
- `GET /api/analytics/top-page` - トップページデータ一括取得
- `GET /api/analytics/monthly` - 月次収支集計
- `GET /api/analytics/sankey` - サンキー図データ
- `GET /api/analytics/balance-sheet` - 貸借対照表データ
- `GET /api/analytics/daily-donations` - 日次寄付金推移

---

### 💵 [05_balance_snapshots.md](./05_balance_snapshots.md)
残高スナップショット管理に関するAPIエンドポイント仕様。

**エンドポイント:**
- `GET /api/balance-snapshots` - 一覧取得（管理者専用）
- `GET /api/balance-snapshots/:id` - 詳細取得（管理者専用）
- `GET /api/balance-snapshots/latest` - 最新残高取得
- `POST /api/balance-snapshots` - 作成（管理者専用）
- `PUT /api/balance-snapshots/:id` - 更新（管理者専用）
- `DELETE /api/balance-snapshots/:id` - 削除（管理者専用）

---

### 🔐 [06_authentication.md](./06_authentication.md)
認証・認可に関するAPIエンドポイント仕様。

**エンドポイント:**
- `POST /api/auth/signup` - ユーザー登録
- `POST /api/auth/login` - ログイン
- `POST /api/auth/logout` - ログアウト
- `POST /api/auth/refresh` - トークンリフレッシュ
- `GET /api/auth/me` - 現在のユーザー情報取得
- `POST /api/auth/change-password` - パスワード変更
- `POST /api/auth/reset-password` - パスワードリセット要求

**認証方式:**
- JWT（JSON Web Token）認証
- Supabase Auth連携
- ロールベースアクセス制御（admin / user）

---

### 📋 [07_endpoint_list.md](./07_endpoint_list.md)
すべてのAPIエンドポイントを一覧表示したリファレンス。

**主な内容:**
- エンドポイント一覧表（認証要件、権限を含む）
- HTTPステータスコード一覧
- レスポンス形式
- ページネーション、ソート、フィルタリング
- レート制限
- CORS設定
- バージョニング

---

## クイックスタート

### 1. 概要を理解する
まず [00_overview.md](./00_overview.md) を読んで、全体像を把握してください。

### 2. データモデルを確認する
[01_data_models.md](./01_data_models.md) でデータ構造を理解してください。

### 3. 必要なAPIを探す
[07_endpoint_list.md](./07_endpoint_list.md) で必要なエンドポイントを見つけてください。

### 4. 詳細仕様を確認する
各機能別ドキュメント（02〜06）で詳細な仕様を確認してください。

---

## APIの構成

### 公開API（認証不要）
一般ユーザー向けのデータ閲覧API：
- 政治団体一覧・詳細
- トランザクション一覧・詳細・CSVエクスポート
- データ集計（月次、サンキー図、貸借対照表、寄付金推移）
- 最新残高取得

### 管理者専用API（認証必須）
データ管理用のAPI：
- 政治団体の作成・更新・削除
- トランザクションのCSVアップロード・プレビュー・削除
- 残高スナップショットの作成・更新・削除
- キャッシュ無効化

### 認証API
ユーザー認証・認可：
- ユーザー登録・ログイン・ログアウト
- トークンリフレッシュ
- パスワード変更・リセット
- ユーザー情報取得

---

## 開発の進め方

### フェーズ1: バックエンドAPI構築
1. プロジェクトセットアップ（Fastify + tRPC）
2. Prismaスキーマの共有化
3. 既存Usecaseの移植
4. APIエンドポイントの実装
5. 認証・認可の実装
6. テストの作成

### フェーズ2: フロントエンド統合
1. APIクライアントの作成
2. webappのServer Actionsを段階的にAPI呼び出しに置き換え
3. adminのServer Actionsを段階的にAPI呼び出しに置き換え
4. エラーハンドリングの統一

### フェーズ3: 最適化・運用
1. キャッシュ戦略の実装（Redis）
2. パフォーマンス最適化
3. モニタリング・ロギング強化
4. CI/CDパイプラインの整備
5. ドキュメントの更新

---

## 技術スタック推奨

### バックエンドフレームワーク
- **Fastify**: 高速で軽量なNode.jsフレームワーク
- **tRPC**: 型安全なRPC、フルスタックTypeScriptに最適

### データベース
- **PostgreSQL**: Supabase経由
- **Prisma ORM**: 既存のスキーマをそのまま活用

### 認証
- **Supabase Auth**: JWT認証、OAuth対応

### キャッシュ
- **Redis**: 高速なインメモリキャッシュ

### デプロイ
- **Vercel / Railway / Fly.io**: サーバーレス or コンテナベース

---

## セキュリティ

### 認証・認可
- JWT認証（Supabase Auth）
- ロールベースアクセス制御
- トークンの有効期限管理
- リフレッシュトークン対応

### データ保護
- HTTPS必須
- 環境変数での秘密情報管理
- SQLインジェクション対策（Prismaで自動対応）
- XSS対策
- CSRF対策

### レート制限
- エンドポイントごとのレート制限
- DDoS対策
- 認証エンドポイントへの厳しい制限

---

## パフォーマンス

### データベース最適化
- 適切なインデックス設定
- IN句を使った複数組織の一括取得
- 不要なJOINの削減

### キャッシュ戦略
- Redis によるキャッシュ
- Cache-Control ヘッダー
- ETag 対応
- 条件付きリクエスト

### レスポンスサイズ削減
- ページネーション必須
- 不要なフィールドの除外
- gzip圧縮

---

## モニタリング

### ログ
- アクセスログ
- エラーログ
- 監査ログ（管理者操作）

### メトリクス
- レスポンスタイム
- エラー率
- リクエスト数
- データベース接続数

### アラート
- エラー率の急上昇
- レスポンスタイムの悪化
- データベース接続エラー

---

## 関連リソース

### 開発環境
- [Prisma スキーマ](../../prisma/schema.prisma)
- [共有モデル](../../shared/models/)
- [既存Usecase](../../webapp/src/server/usecases/)

### 外部ドキュメント
- [Fastify Documentation](https://www.fastify.io/)
- [tRPC Documentation](https://trpc.io/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Supabase Auth Documentation](https://supabase.com/docs/guides/auth)

---

## 貢献

このAPIドキュメントは実装と並行して更新されます。

### ドキュメント更新時
1. 実装とドキュメントの整合性を保つ
2. 変更があった場合は関連ドキュメントも更新
3. エンドポイント追加時は一覧表も更新

### 質問・提案
GitHub Issuesで質問や提案を受け付けています。

---

## ライセンス

このドキュメントは [GNU Affero General Public License v3.0](../../LICENSE) の下でライセンスされています。
