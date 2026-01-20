---
name: tdd-workflow
description: 新機能を書いたり、バグを修正したり、コードをリファクタリングする際に使用します。80%以上のカバレッジを含むテスト駆動開発を強制します（単体テスト、統合テスト、E2Eテスト）。
---

# テスト駆動開発ワークフロー

このスキルにより、すべてのコード開発がTDD原則に従い、包括的なテストカバレッジを確保します。

## 使用する場合

- 新しい機能または機能を実装する場合
- バグまたは問題を修正する場合
- 既存コードをリファクタリングする場合
- APIエンドポイントを追加する場合
- 新しいコンポーネントを作成する場合

## コア原則

### 1. テスト優先

常にテストを最初に書いてから、テストをパスするコードを実装します。

### 2. カバレッジ要件

- 最小80%カバレッジ（単体 + 統合 + E2E）
- すべてのエッジケースをカバー
- エラーシナリオのテスト
- 境界条件の検証

### 3. テストの種類

#### 単体テスト
- 個々の関数とユーティリティ
- コンポーネントロジック
- 純粋な関数
- ヘルパーとユーティリティ

#### 統合テスト
- APIエンドポイント
- データベース操作
- サービス間の相互作用
- 外部API呼び出し

#### E2Eテスト（Playwright）
- 重要なユーザーフロー
- 完全なワークフロー
- ブラウザ自動化
- UIインタラクション

## TDDワークフロー手順

### ステップ1：ユーザージャーニーを記述

```
[ロール]として、[アクション]したいので、[利点]を得られる

例：
ユーザーとして、市場をセマンティック検索したいので、正確なキーワードなしで関連する市場を見つけられる。
```

### ステップ2：テストケースを生成

各ユーザージャーニーについて、包括的なテストケースを作成します：

```typescript
describe('セマンティック検索', () => {
  it('クエリに関連する市場を返す', async () => {
    // テスト実装
  })

  it('空のクエリを適切に処理する', async () => {
    // テストエッジケース
  })

  it('Redisが利用できない場合、部分文字列検索にフォールバックする', async () => {
    // テストフォールバック動作
  })

  it('結果を類似度スコアでソートする', async () => {
    // テストソートロジック
  })
})
```

### ステップ3：テストを実行（失敗するはず）

```bash
npm test
# テストは失敗するはず - まだ実装していないため
```

### ステップ4：コードを実装

テストをパスさせるための最小限のコードを記述します：

```typescript
// テストによってガイドされた実装
export async function searchMarkets(query: string) {
  // ここに実装
}
```

### ステップ5：テストを再度実行

```bash
npm test
# テストはパスするはず
```

### ステップ6：リファクタリング

テストが緑のまま、コード品質を改善します：
- 重複を削除
- ネーミングを改善
- パフォーマンスを最適化
- 可読性を向上

### ステップ7：カバレッジを確認

```bash
npm run test:coverage
# 80%以上のカバレッジが達成されたことを確認
```

## テストパターン

### 単体テストパターン（Jest/Vitest）

```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('ボタンコンポーネント', () => {
  it('正しいテキストでレンダリングされる', () => {
    render(<Button>クリック</Button>)
    expect(screen.getByText('クリック')).toBeInTheDocument()
  })

  it('クリックされるとonClickが呼ばれる', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>クリック</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('disabledプロップがtrueの場合は無効になる', () => {
    render(<Button disabled>クリック</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### API統合テストパターン

```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('市場を正常に返す', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('クエリパラメータを検証する', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('データベースエラーを適切に処理する', async () => {
    // データベース障害をモック
    const request = new NextRequest('http://localhost/api/markets')
    // エラー処理をテスト
  })
})
```

### E2Eテストパターン（Playwright）

```typescript
import { test, expect } from '@playwright/test'

test('ユーザーが市場を検索およびフィルタリングできる', async ({ page }) => {
  // 市場ページに移動
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // ページが読み込まれたことを確認
  await expect(page.locator('h1')).toContainText('Markets')

  // 市場を検索
  await page.fill('input[placeholder="Search markets"]', 'election')

  // デバウンスと結果を待つ
  await page.waitForTimeout(600)

  // 検索結果が表示されることを確認
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 結果に検索語が含まれることを確認
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // ステータスでフィルタリング
  await page.click('button:has-text("Active")')

  // フィルタリングされた結果を確認
  await expect(results).toHaveCount(3)
})

test('ユーザーが新しい市場を作成できる', async ({ page }) => {
  // まずログイン
  await page.goto('/creator-dashboard')

  // 市場作成フォームに入力
  await page.fill('input[name="name"]', 'Test Market')
  await page.fill('textarea[name="description"]', 'テスト説明')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // フォームを送信
  await page.click('button[type="submit"]')

  // 成功メッセージを確認
  await expect(page.locator('text=市場が正常に作成されました')).toBeVisible()

  // 市場ページへのリダイレクトを確認
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## テストファイル構成

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # 単体テスト
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # 統合テスト
└── e2e/
    ├── markets.spec.ts               # E2Eテスト
    ├── trading.spec.ts
    └── auth.spec.ts
```

## 外部サービスのモック

### Supabaseモック

```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: [{ id: 1, name: 'Test Market' }],
          error: null
        }))
      }))
    }))
  }
}))
```

### Redisモック

```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ])),
  checkRedisHealth: jest.fn(() => Promise.resolve({ connected: true }))
}))
```

### OpenAIモック

```typescript
jest.mock('@/lib/openai', () => ({
  generateEmbedding: jest.fn(() => Promise.resolve(
    new Array(1536).fill(0.1) // モック1536次元埋め込み
  ))
}))
```

## テストカバレッジ検証

### カバレッジレポートを実行

```bash
npm run test:coverage
```

### カバレッジしきい値

```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## よくあるテストの間違い

### ❌ 間違い：実装の詳細をテスト

```typescript
// 内部状態をテストしない
expect(component.state.count).toBe(5)
```

### ✅ 正解：ユーザーが見える動作をテスト

```typescript
// ユーザーが見るものをテスト
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### ❌ 間違い：もろいセレクタ

```typescript
// 簡単に壊れる
await page.click('.css-class-xyz')
```

### ✅ 正解：セマンティックなセレクタ

```typescript
// 変更に強い
await page.click('button:has-text("送信")')
await page.click('[data-testid="submit-button"]')
```

### ❌ 間違い：テスト分離なし

```typescript
// テストが相互に依存
test('ユーザーを作成', () => { /* ... */ })
test('同じユーザーを更新', () => { /* 前のテストに依存 */ })
```

### ✅ 正解：独立したテスト

```typescript
// 各テストが独自にデータを設定
test('ユーザーを作成', () => {
  const user = createTestUser()
  // テストロジック
})

test('ユーザーを更新', () => {
  const user = createTestUser()
  // 更新ロジック
})
```

## 継続的なテスト

### 開発中のウォッチモード

```bash
npm test -- --watch
# ファイル変更時にテストが自動実行される
```

### プリコミットフック

```bash
# 各コミット前に実行
npm test && npm run lint
```

### CI/CD統合

```yaml
# GitHub Actions
- name: テストを実行
  run: npm test -- --coverage
- name: カバレッジをアップロード
  uses: codecov/codecov-action@v3
```

## ベストプラクティス

1. **テストを最初に書く** - 常にTDD
2. **1テスト1アサーション** - 単一の動作に焦点
3. **説明的なテスト名** - テスト内容を説明
4. **Arrange-Act-Assert** - 明確なテスト構造
5. **外部依存をモック** - 単体テストを分離
6. **エッジケースをテスト** - null、undefined、空、大きな値
7. **エラーパスをテスト** - ハッピーパスだけでなく
8. **テストを高速に** - 単体テスト < 50msごと
9. **テスト後にクリーンアップ** - 副作用なし
10. **カバレッジレポートを確認** - ギャップを特定

## 成功指標

- 80%以上のコードカバレッジを達成
- すべてのテストがパス（緑）
- スキップまたは無効化されたテストなし
- テスト実行が高速（単体テスト < 30秒）
- 重要なユーザーフローをカバーするE2Eテスト
- テストが本番前にバグをキャッチ

---

**覚えておいてください**：テストはオプションではありません。これは自信を持ったリファクタリング、迅速な開発、および本番環境の信頼性を実現するための安全ネットです。
