---
name: security-review
description: 認証、ユーザー入力の処理、シークレットの扱い、APIエンドポイント作成、決済・機密機能の実装時に使用します。包括的なセキュリティチェックリストとパターンを提供します。
---

# セキュリティレビュースキル

このスキルはすべてのコードがセキュリティベストプラクティスに従い、潜在的な脆弱性がないことを確認します。

## アクティベーション時

- 認証または認可の実装
- ユーザー入力またはファイルアップロードの処理
- 新しいAPIエンドポイントの作成
- シークレットまたは認証情報の扱い
- 決済機能の実装
- 機密データの保存または送信
- サードパーティAPIの統合

## セキュリティチェックリスト

### 1. シークレット管理

#### ❌ してはいけないこと
```typescript
const apiKey = "sk-proj-xxxxx"  // ハードコードされたシークレット
const dbPassword = "password123" // ソースコード内
```

#### ✅ いつもやるべきこと
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// シークレットが存在することを確認
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 確認ステップ
- [ ] ハードコードされたAPIキー、トークン、パスワードがない
- [ ] すべてのシークレットが環境変数に設定されている
- [ ] `.env.local`が.gitignoreに含まれている
- [ ] gitの履歴にシークレットが残っていない
- [ ] 本番環境のシークレットがホスティングプラットフォーム（Vercel、Railway）に設定されている

### 2. 入力検証

#### ユーザー入力は常に検証

```typescript
import { z } from 'zod'

// 検証スキーマを定義
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 処理前に検証
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### ファイルアップロード検証
```typescript
function validateFileUpload(file: File) {
  // サイズチェック（最大5MB）
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // タイプチェック
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // 拡張子チェック
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

#### 確認ステップ
- [ ] すべてのユーザー入力がスキーマで検証されている
- [ ] ファイルアップロードが制限されている（サイズ、タイプ、拡張子）
- [ ] ユーザー入力がクエリで直接使用されていない
- [ ] ホワイトリスト検証を使用している（ブラックリストではない）
- [ ] エラーメッセージが機密情報を漏らしていない

### 3. SQLインジェクション対策

#### ❌ SQLを連結してはいけない
```typescript
// 危険 - SQLインジェクション脆弱性
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### ✅ パラメータ化されたクエリを常に使用
```typescript
// 安全 - パラメータ化されたクエリ
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// または生のSQLで
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 確認ステップ
- [ ] すべてのデータベースクエリがパラメータ化されている
- [ ] SQLに文字列連結がない
- [ ] ORM/クエリビルダーが正しく使用されている
- [ ] Supabaseクエリが適切にサニタイズされている

### 4. 認証と認可

#### JWTトークン処理
```typescript
// ❌ 間違い：localStorage（XSS脆弱性）
localStorage.setItem('token', token)

// ✅ 正しい：httpOnlyクッキー
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 認可チェック
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 常に最初に認可を確認
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  // 削除に進む
  await db.users.delete({ where: { id: userId } })
}
```

#### 行レベルセキュリティ（Supabase）
```sql
-- すべてのテーブルでRLSを有効化
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- ユーザーは自分のデータのみ表示できる
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- ユーザーは自分のデータのみ更新できる
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### 確認ステップ
- [ ] トークンがhttpOnlyクッキーに格納されている（localStorageではない）
- [ ] 機密操作前に認可チェックが実装されている
- [ ] Supabaseで行レベルセキュリティが有効化されている
- [ ] ロールベースのアクセス制御が実装されている
- [ ] セッション管理が安全である

### 5. XSS対策

#### HTMLをサニタイズ
```typescript
import DOMPurify from 'isomorphic-dompurify'

// ユーザーが提供したHTMLは常にサニタイズ
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### コンテンツセキュリティポリシー
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

#### 確認ステップ
- [ ] ユーザーが提供したHTMLがサニタイズされている
- [ ] CSPヘッダーが設定されている
- [ ] 検証されていない動的コンテンツがレンダリングされていない
- [ ] Reactの組み込みXSS保護が使用されている

### 6. CSRF対策

#### CSRFトークン
```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    )
  }

  // リクエストを処理
}
```

#### SameSiteクッキー
```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### 確認ステップ
- [ ] 状態を変更する操作にCSRFトークンがある
- [ ] すべてのクッキーにSameSite=Strictが設定されている
- [ ] ダブルサブミットクッキーパターンが実装されている

### 7. レート制限

#### APIレート制限
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15分
  max: 100, // ウィンドウあたり100リクエスト
  message: 'Too many requests'
})

// ルートに適用
app.use('/api/', limiter)
```

#### 高コスト操作
```typescript
// 検索に対するアグレッシブなレート制限
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1分
  max: 10, // 1分あたり10リクエスト
  message: 'Too many search requests'
})

app.use('/api/search', searchLimiter)
```

#### 確認ステップ
- [ ] すべてのAPIエンドポイントにレート制限がある
- [ ] 高コスト操作に対して厳しい制限がある
- [ ] IPベースのレート制限がある
- [ ] ユーザーベースのレート制限がある（認証済み）

### 8. 機密データの露出

#### ログ
```typescript
// ❌ 間違い：機密データをログに出力
console.log('User login:', { email, password })
console.log('Payment:', { cardNumber, cvv })

// ✅ 正しい：機密データをマスク
console.log('User login:', { email, userId })
console.log('Payment:', { last4: card.last4, userId })
```

#### エラーメッセージ
```typescript
// ❌ 間違い：内部詳細を露出
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ 正しい：汎用エラーメッセージ
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

#### 確認ステップ
- [ ] パスワード、トークン、シークレットがログに含まれていない
- [ ] ユーザー向けのエラーメッセージが汎用的である
- [ ] 詳細なエラーはサーバーログのみに記録されている
- [ ] スタックトレースがユーザーに露出していない

### 9. ブロックチェーンセキュリティ（Solana）

#### ウォレット検証
```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

#### トランザクション検証
```typescript
async function verifyTransaction(transaction: Transaction) {
  // 受信者を確認
  if (transaction.to !== expectedRecipient) {
    throw new Error('Invalid recipient')
  }

  // 金額を確認
  if (transaction.amount > maxAmount) {
    throw new Error('Amount exceeds limit')
  }

  // ユーザーが十分な残高を持つことを確認
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('Insufficient balance')
  }

  return true
}
```

#### 確認ステップ
- [ ] ウォレット署名が検証されている
- [ ] トランザクションの詳細が検証されている
- [ ] トランザクション前に残高チェックが行われている
- [ ] トランザクション署名が無条件に実行されていない

### 10. 依存関係のセキュリティ

#### 定期的な更新
```bash
# 脆弱性をチェック
npm audit

# 自動修正可能な問題を修正
npm audit fix

# 依存関係を更新
npm update

# 古いパッケージを確認
npm outdated
```

#### ロックファイル
```bash
# ロックファイルは必ずコミット
git add package-lock.json

# CI/CDで再現可能なビルドのために使用
npm ci  # npm installの代わり
```

#### 確認ステップ
- [ ] 依存関係が最新である
- [ ] 既知の脆弱性がない（npm auditがクリーン）
- [ ] ロックファイルがコミットされている
- [ ] GitHubでDependabotが有効化されている
- [ ] セキュリティ更新が定期的に行われている

## セキュリティテスト

### 自動セキュリティテスト
```typescript
// 認証をテスト
test('requires authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// 認可をテスト
test('requires admin role', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// 入力検証をテスト
test('rejects invalid input', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// レート制限をテスト
test('enforces rate limits', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )

  const responses = await Promise.all(requests)
  const tooManyRequests = responses.filter(r => r.status === 429)

  expect(tooManyRequests.length).toBeGreaterThan(0)
})
```

## 本番環境デプロイ前のセキュリティチェックリスト

ANY本番環境へのデプロイ前に確認してください：

- [ ] **シークレット**：ハードコードされたシークレットがなく、すべて環境変数に設定
- [ ] **入力検証**：すべてのユーザー入力が検証されている
- [ ] **SQLインジェクション**：すべてのクエリがパラメータ化されている
- [ ] **XSS**：ユーザーコンテンツがサニタイズされている
- [ ] **CSRF**：保護が有効化されている
- [ ] **認証**：適切なトークン処理が実装されている
- [ ] **認可**：ロールチェックが実装されている
- [ ] **レート制限**：すべてのエンドポイントで有効化されている
- [ ] **HTTPS**：本番環境で強制されている
- [ ] **セキュリティヘッダー**：CSP、X-Frame-Optionsが設定されている
- [ ] **エラーハンドリング**：エラーに機密データが含まれていない
- [ ] **ログ**：機密データがログに出力されていない
- [ ] **依存関係**：最新で脆弱性がない
- [ ] **行レベルセキュリティ**：Supabaseで有効化されている
- [ ] **CORS**：適切に設定されている
- [ ] **ファイルアップロード**：検証されている（サイズ、タイプ）
- [ ] **ウォレット署名**：検証されている（ブロックチェーンの場合）

## リソース

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**重要**：セキュリティはオプションではありません。1つの脆弱性がプラットフォーム全体を侵害する可能性があります。疑わしい場合は、安全側を選択してください。
