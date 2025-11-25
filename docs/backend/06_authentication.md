# 認証API

ユーザー認証と認可に関するAPIエンドポイント。Supabase Authを利用したJWT認証を実装します。

---

## 認証方式

### JWT（JSON Web Token）認証
- Supabase Authが発行するJWTトークンを使用
- トークンはHTTPヘッダー `Authorization: Bearer {token}` で送信
- トークンの有効期限は1時間（Supabaseのデフォルト設定）
- リフレッシュトークンで自動更新可能

### ロール
```typescript
enum UserRole {
  admin = "admin",         // 管理者（全権限）
  user = "user"            // 一般ユーザー（閲覧のみ）
}
```

---

## エンドポイント一覧

### 1. ユーザー登録（サインアップ）

新しいユーザーを登録します。

**エンドポイント**
```
POST /api/auth/signup
```

**認証**: 不要

**リクエストボディ**
```typescript
{
  email: string;           // 必須: メールアドレス
  password: string;        // 必須: パスワード（8文字以上）
  role?: UserRole;         // 任意: ユーザーロール（デフォルト: user）
}
```

**リクエスト例**
```json
{
  "email": "admin@example.com",
  "password": "SecurePassword123!",
  "role": "admin"
}
```

**レスポンス**
```typescript
{
  user: {
    id: string;
    email: string;
    role: UserRole;
    createdAt: string;
  };
  session: {
    accessToken: string;
    refreshToken: string;
    expiresIn: number;
  };
}
```

**レスポンス例**
```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "admin@example.com",
    "role": "admin",
    "createdAt": "2024-01-01T00:00:00.000Z"
  },
  "session": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "v1.MjAyNC0wMS0wMVQwMDowMDowMC4wMDBa...",
    "expiresIn": 3600
  }
}
```

**ステータスコード**
- `201 Created`: 登録成功
- `400 Bad Request`: バリデーションエラー（メールアドレス形式不正、パスワード要件未満など）
- `409 Conflict`: メールアドレスが既に登録済み
- `500 Internal Server Error`: サーバーエラー

**バリデーションルール**
- `email`: 必須、有効なメールアドレス形式
- `password`: 必須、8文字以上、英数字と記号を含む
- `role`: 任意、`"admin"` または `"user"`（デフォルト: `"user"`）

**注意事項**
- メール確認が必要な場合は、Supabaseの設定に従う
- 初回ユーザー登録時は管理者権限の付与を検討
- 本番環境では招待制などの追加制限を推奨

---

### 2. ログイン

メールアドレスとパスワードでログインします。

**エンドポイント**
```
POST /api/auth/login
```

**認証**: 不要

**リクエストボディ**
```typescript
{
  email: string;           // 必須: メールアドレス
  password: string;        // 必須: パスワード
}
```

**リクエスト例**
```json
{
  "email": "admin@example.com",
  "password": "SecurePassword123!"
}
```

**レスポンス**
```typescript
{
  user: {
    id: string;
    email: string;
    role: UserRole;
    createdAt: string;
  };
  session: {
    accessToken: string;
    refreshToken: string;
    expiresIn: number;
  };
}
```

**レスポンス例**
```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "admin@example.com",
    "role": "admin",
    "createdAt": "2024-01-01T00:00:00.000Z"
  },
  "session": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "v1.MjAyNC0wMS0wMVQwMDowMDowMC4wMDBa...",
    "expiresIn": 3600
  }
}
```

**ステータスコード**
- `200 OK`: ログイン成功
- `400 Bad Request`: バリデーションエラー
- `401 Unauthorized`: メールアドレスまたはパスワードが不正
- `500 Internal Server Error`: サーバーエラー

---

### 3. ログアウト

現在のセッションを終了します。

**エンドポイント**
```
POST /api/auth/logout
```

**認証**: 必須

**リクエストボディ**
なし

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
  "message": "Logged out successfully"
}
```

**ステータスコード**
- `200 OK`: ログアウト成功
- `401 Unauthorized`: 認証エラー
- `500 Internal Server Error`: サーバーエラー

---

### 4. トークンリフレッシュ

リフレッシュトークンを使用して新しいアクセストークンを取得します。

**エンドポイント**
```
POST /api/auth/refresh
```

**認証**: 不要（リフレッシュトークンが必要）

**リクエストボディ**
```typescript
{
  refreshToken: string;    // 必須: リフレッシュトークン
}
```

**リクエスト例**
```json
{
  "refreshToken": "v1.MjAyNC0wMS0wMVQwMDowMDowMC4wMDBa..."
}
```

**レスポンス**
```typescript
{
  session: {
    accessToken: string;
    refreshToken: string;
    expiresIn: number;
  };
}
```

**レスポンス例**
```json
{
  "session": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "v1.MjAyNC0wMS0wMVQwMDowMDowMC4wMDBa...",
    "expiresIn": 3600
  }
}
```

**ステータスコード**
- `200 OK`: トークンリフレッシュ成功
- `400 Bad Request`: リフレッシュトークンが未指定
- `401 Unauthorized`: リフレッシュトークンが無効または期限切れ
- `500 Internal Server Error`: サーバーエラー

---

### 5. 現在のユーザー情報取得

ログイン中のユーザー情報を取得します。

**エンドポイント**
```
GET /api/auth/me
```

**認証**: 必須

**レスポンス**
```typescript
{
  user: {
    id: string;
    email: string;
    role: UserRole;
    createdAt: string;
    updatedAt: string;
  };
}
```

**レスポンス例**
```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "admin@example.com",
    "role": "admin",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-15T12:30:00.000Z"
  }
}
```

**ステータスコード**
- `200 OK`: 取得成功
- `401 Unauthorized`: 認証エラー
- `500 Internal Server Error`: サーバーエラー

---

### 6. パスワード変更

ログイン中のユーザーのパスワードを変更します。

**エンドポイント**
```
POST /api/auth/change-password
```

**認証**: 必須

**リクエストボディ**
```typescript
{
  currentPassword: string;  // 必須: 現在のパスワード
  newPassword: string;      // 必須: 新しいパスワード（8文字以上）
}
```

**リクエスト例**
```json
{
  "currentPassword": "OldPassword123!",
  "newPassword": "NewSecurePassword456!"
}
```

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
  "message": "Password changed successfully"
}
```

**ステータスコード**
- `200 OK`: 変更成功
- `400 Bad Request`: バリデーションエラー（新しいパスワードが要件を満たさないなど）
- `401 Unauthorized`: 認証エラーまたは現在のパスワードが不正
- `500 Internal Server Error`: サーバーエラー

---

### 7. パスワードリセット要求

パスワードリセット用のメールを送信します。

**エンドポイント**
```
POST /api/auth/reset-password
```

**認証**: 不要

**リクエストボディ**
```typescript
{
  email: string;           // 必須: メールアドレス
}
```

**リクエスト例**
```json
{
  "email": "admin@example.com"
}
```

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
  "message": "Password reset email sent"
}
```

**ステータスコード**
- `200 OK`: メール送信成功（メールアドレスが存在しない場合も200を返す）
- `400 Bad Request`: バリデーションエラー
- `500 Internal Server Error`: サーバーエラー

**注意事項**
- セキュリティのため、メールアドレスの存在確認結果は返さない
- パスワードリセットリンクの有効期限は24時間（Supabaseのデフォルト設定）

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
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid email or password"
  }
}
```

**エラーコード一覧**
- `INVALID_CREDENTIALS`: メールアドレスまたはパスワードが不正
- `USER_EXISTS`: ユーザーが既に存在
- `INVALID_TOKEN`: トークンが無効
- `TOKEN_EXPIRED`: トークンの有効期限切れ
- `UNAUTHORIZED`: 認証エラー
- `FORBIDDEN`: 権限不足
- `VALIDATION_ERROR`: バリデーションエラー

---

## 使用例（フロントエンド）

### ログイン
```typescript
const response = await fetch('/api/auth/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'admin@example.com',
    password: 'SecurePassword123!'
  })
});
const { user, session } = await response.json();

// トークンを保存（localStorage / sessionStorage / cookie）
localStorage.setItem('accessToken', session.accessToken);
localStorage.setItem('refreshToken', session.refreshToken);
```

### 認証が必要なAPIリクエスト
```typescript
const token = localStorage.getItem('accessToken');
const response = await fetch('/api/political-organizations', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    displayName: '田中花子',
    slug: 'tanaka-hanako'
  })
});
```

### トークンリフレッシュ
```typescript
const refreshToken = localStorage.getItem('refreshToken');
const response = await fetch('/api/auth/refresh', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    refreshToken
  })
});
const { session } = await response.json();
localStorage.setItem('accessToken', session.accessToken);
localStorage.setItem('refreshToken', session.refreshToken);
```

### ログアウト
```typescript
const token = localStorage.getItem('accessToken');
await fetch('/api/auth/logout', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
localStorage.removeItem('accessToken');
localStorage.removeItem('refreshToken');
```

---

## ミドルウェア実装

### 認証ミドルウェア

すべての保護されたエンドポイントに適用するミドルウェア。

```typescript
import { createClient } from '@supabase/supabase-js';

export async function authMiddleware(req: Request) {
  const authHeader = req.headers.get('Authorization');
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    throw new Error('Unauthorized');
  }
  
  const token = authHeader.substring(7);
  const supabase = createClient(
    process.env.SUPABASE_URL!,
    process.env.SUPABASE_ANON_KEY!
  );
  
  const { data: { user }, error } = await supabase.auth.getUser(token);
  
  if (error || !user) {
    throw new Error('Invalid token');
  }
  
  return user;
}
```

### 認可ミドルウェア（管理者チェック）

管理者専用エンドポイントに適用するミドルウェア。

```typescript
export async function requireAdmin(req: Request) {
  const user = await authMiddleware(req);
  
  // データベースからユーザーロールを取得
  const dbUser = await prisma.user.findUnique({
    where: { authId: user.id }
  });
  
  if (!dbUser || dbUser.role !== 'admin') {
    throw new Error('Forbidden: Admin access required');
  }
  
  return dbUser;
}
```

---

## セキュリティベストプラクティス

### トークンの保存
- **推奨**: HttpOnly Cookieに保存（XSS対策）
- **非推奨**: localStorage / sessionStorage（XSS脆弱性）

### HTTPS必須
- 本番環境ではHTTPS通信を必須とする
- トークンの盗聴を防ぐ

### CORS設定
```typescript
const allowedOrigins = [
  'https://webapp.example.com',
  'https://admin.example.com'
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true
}));
```

### レート制限
```typescript
// ログインエンドポイントに対して厳しいレート制限
app.use('/api/auth/login', rateLimit({
  windowMs: 15 * 60 * 1000, // 15分
  max: 5                     // 最大5回
}));

// その他の認証エンドポイント
app.use('/api/auth', rateLimit({
  windowMs: 15 * 60 * 1000, // 15分
  max: 20                    // 最大20回
}));
```

### パスワードポリシー
- 最小8文字
- 英大文字・小文字・数字・記号を含む
- 一般的なパスワードを禁止（辞書攻撃対策）
- パスワード履歴チェック（過去のパスワードを再利用不可）

---

## Supabase Auth連携

### 環境変数
```env
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

### Supabase Clientの初期化
```typescript
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
);

// サーバーサイドでのユーザー管理用（管理者権限）
export const supabaseAdmin = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);
```

### ユーザー登録時の処理
1. Supabase Authにユーザー作成
2. データベースの `users` テーブルにユーザー情報を保存
3. ロールを設定（デフォルト: `user`）

```typescript
// Supabase Authにユーザー作成
const { data: authData, error: authError } = await supabaseAdmin.auth.admin.createUser({
  email,
  password,
  email_confirm: true
});

if (authError) throw authError;

// データベースにユーザー情報を保存
const user = await prisma.user.create({
  data: {
    authId: authData.user.id,
    email,
    role: role || 'user'
  }
});
```

---

## 今後の拡張

- OAuth連携（Google、GitHub など）
- 二要素認証（2FA）
- セッション管理（複数デバイス対応）
- 監査ログ（ログイン履歴、アクセス履歴）
- IP制限（管理画面へのアクセス制限）
- メール認証必須化
