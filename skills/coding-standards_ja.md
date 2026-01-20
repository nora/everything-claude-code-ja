---
name: coding-standards
description: TypeScript、JavaScript、React、Node.js開発に適用される普遍的なコード規格、ベストプラクティス、パターン。
---

# コード規格とベストプラクティス

すべてのプロジェクトに適用される普遍的なコード規格。

## コード品質の原則

### 1. 読みやすさを最優先
- コードは書くより読む頻度が高い
- 明確な変数名と関数名
- コメントより自己説明的なコードが好ましい
- 一貫した形式

### 2. KISS（Keep It Simple, Stupid）
- 動作する最もシンプルなソリューション
- 過度なエンジニアリングを避ける
- 性急な最適化をしない
- 複雑さより理解しやすさを優先

### 3. DRY（Don't Repeat Yourself）
- 共通ロジックを関数に抽出する
- 再利用可能なコンポーネントを作成する
- モジュール間でユーティリティを共有する
- コピペプログラミングを避ける

### 4. YAGNI（You Aren't Gonna Need It）
- 必要になるまで機能を実装しない
- 推測的な汎用性を避ける
- 必要な場合のみ複雑さを追加する
- シンプルに始めて、必要に応じてリファクタリング

## TypeScript/JavaScript規格

### 変数名

```typescript
// ✅ 良好例：説明的な名前
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 不良例：不明確な名前
const q = 'election'
const flag = true
const x = 1000
```

### 関数名

```typescript
// ✅ 良好例：動詞-名詞パターン
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 不良例：不明確または名詞のみ
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### イミュータビリティパターン（重要）

```typescript
// ✅ スプレッド演算子を常に使用
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 直接ミューテーションは禁止
user.name = 'New Name'  // 不良
items.push(newItem)     // 不良
```

### エラーハンドリング

```typescript
// ✅ 良好例：包括的なエラーハンドリング
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}

// ❌ 不良例：エラーハンドリングなし
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Awaitベストプラクティス

```typescript
// ✅ 良好例：可能な場合は並列実行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 不良例：不要な順序実行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 型安全

```typescript
// ✅ 良好例：適切な型
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // 実装
}

// ❌ 不良例：'any'を使用
function getMarket(id: any): Promise<any> {
  // 実装
}
```

## React ベストプラクティス

### コンポーネント構造

```typescript
// ✅ 良好例：型付き関数コンポーネント
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// ❌ 不良例：型なし、不明確な構造
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### カスタムフック

```typescript
// ✅ 良好例：再利用可能なカスタムフック
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// 使用法
const debouncedQuery = useDebounce(searchQuery, 500)
```

### ステート管理

```typescript
// ✅ 良好例：適切なステート更新
const [count, setCount] = useState(0)

// 前のステートに基づくステート更新の場合は関数型更新を使用
setCount(prev => prev + 1)

// ❌ 不良例：ステートの直接参照
setCount(count + 1)  // 非同期シナリオで古い値になる可能性あり
```

### 条件付きレンダリング

```typescript
// ✅ 良好例：明確な条件付きレンダリング
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 不良例：三項演算子の過度な使用
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API設計規格

### REST API慣例

```
GET    /api/markets              # すべてのマーケットを一覧表示
GET    /api/markets/:id          # 特定のマーケットを取得
POST   /api/markets              # 新規マーケットを作成
PUT    /api/markets/:id          # マーケットを完全更新
PATCH  /api/markets/:id          # マーケットを部分更新
DELETE /api/markets/:id          # マーケットを削除

# フィルタリング用クエリパラメータ
GET /api/markets?status=active&limit=10&offset=0
```

### レスポンス形式

```typescript
// ✅ 良好例：一貫したレスポンス構造
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}

// 成功レスポンス
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// エラーレスポンス
return NextResponse.json({
  success: false,
  error: 'Invalid request'
}, { status: 400 })
```

### 入力バリデーション

```typescript
import { z } from 'zod'

// ✅ 良好例：スキーマバリデーション
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1)
})

export async function POST(request: Request) {
  const body = await request.json()

  try {
    const validated = CreateMarketSchema.parse(body)
    // バリデーション済みデータで処理を進める
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: 'Validation failed',
        details: error.errors
      }, { status: 400 })
    }
  }
}
```

## ファイル構成

### プロジェクト構造

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # APIルート
│   ├── markets/           # マーケットページ
│   └── (auth)/           # 認証ページ（ルートグループ）
├── components/            # Reactコンポーネント
│   ├── ui/               # 汎用UIコンポーネント
│   ├── forms/            # フォームコンポーネント
│   └── layouts/          # レイアウトコンポーネント
├── hooks/                # カスタムReactフック
├── lib/                  # ユーティリティと設定
│   ├── api/             # APIクライアント
│   ├── utils/           # ヘルパー関数
│   └── constants/       # 定数
├── types/                # TypeScript型定義
└── styles/              # グローバルスタイル
```

### ファイル名

```
components/Button.tsx          # コンポーネント：PascalCase
hooks/useAuth.ts              # フック：camelCase 'use'プレフィックス付き
lib/formatDate.ts             # ユーティリティ：camelCase
types/market.types.ts         # 型定義：camelCase .typesサフィックス付き
```

## コメントとドキュメント

### コメントを書く場合

```typescript
// ✅ 良好例：「何」ではなく「なぜ」を説明
// 停止時にAPI を圧倒することを回避するため指数バックオフを使用
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 大きな配列でのパフォーマンス最適化のため意図的にミューテーションを使用
items.push(newItem)

// ❌ 不良例：自明なことを述べる
// カウンターを1増やす
count++

// 名前をユーザーの名前に設定
name = user.name
```

### 公開APIのJSDoc

```typescript
/**
 * セマンティック類似度を使用してマーケットを検索します。
 *
 * @param query - 自然言語検索クエリ
 * @param limit - 最大結果数（デフォルト：10）
 * @returns 類似度スコアで並べ替えたマーケットの配列
 * @throws {Error} OpenAI API失敗またはRedis未利用可能の場合
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('election', 5)
 * console.log(results[0].name) // "Trump vs Biden"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10
): Promise<Market[]> {
  // 実装
}
```

## パフォーマンスベストプラクティス

### メモ化

```typescript
import { useMemo, useCallback } from 'react'

// ✅ 良好例：高コストな計算をメモ化
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 良好例：コールバックをメモ化
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 遅延読み込み

```typescript
import { lazy, Suspense } from 'react'

// ✅ 良好例：重いコンポーネントを遅延読み込み
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### データベースクエリ

```typescript
// ✅ 良好例：必要な列のみを選択
const { data } = await supabase
  .from('markets')
  .select('id, name, status')
  .limit(10)

// ❌ 不良例：すべてを選択
const { data } = await supabase
  .from('markets')
  .select('*')
```

## テスト規格

### テスト構造（AAAパターン）

```typescript
test('cos類似度を正しく計算', () => {
  // 準備（Arrange）
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // 実行（Act）
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // 検証（Assert）
  expect(similarity).toBe(0)
})
```

### テスト名

```typescript
// ✅ 良好例：説明的なテスト名
test('クエリに一致するマーケットがない場合、空配列を返す', () => { })
test('OpenAI APIキーが見つからない場合、エラーをスロー', () => { })
test('Redis が利用できない場合、部分文字列検索にフォールバック', () => { })

// ❌ 不良例：曖昧なテスト名
test('動作する', () => { })
test('検索テスト', () => { })
```

## コード臭検出

これらのアンチパターンに注意してください：

### 1. 長い関数
```typescript
// ❌ 不良例：関数が50行以上
function processMarketData() {
  // 100行のコード
}

// ✅ 良好例：小さな関数に分割
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 深いネスト
```typescript
// ❌ 不良例：5段階以上のネスト
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 何かを実行
        }
      }
    }
  }
}

// ✅ 良好例：早期リターン
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 何かを実行
```

### 3. マジックナンバー
```typescript
// ❌ 不良例：説明のない数字
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ 良好例：名前付き定数
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**覚えておいてください**：コード品質は譲歩できません。明確で保守しやすいコードは、迅速な開発と自信を持ったリファクタリングを実現します。
