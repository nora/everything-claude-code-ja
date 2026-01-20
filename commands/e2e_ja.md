---
description: Playwrightを使用してエンドツーエンドテストを生成・実行します。テストジャーニーを作成し、テストを実行し、スクリーンショット/ビデオ/トレースをキャプチャして成果物をアップロードします。
---

# E2E コマンド

このコマンドは**e2e-runner**エージェントを呼び出して、Playwrightを使用したエンドツーエンドテストの生成、保守、実行を行います。

## このコマンドが行うこと

1. **テストジャーニーを生成** - ユーザーフロー用のPlaywrightテストを作成
2. **E2Eテストを実行** - 複数のブラウザ間でテストを実行
3. **成果物をキャプチャ** - 失敗時のスクリーンショット、ビデオ、トレース
4. **結果をアップロード** - HTMLレポートとJUnit XML
5. **不安定なテストを特定** - 不安定なテストを隔離

## 使用時期

以下の場合に `/e2e` を使用します:
- 重要なユーザージャーニー（ログイン、トレード、支払い）をテスト
- マルチステップフローがエンドツーエンドで機能することを確認
- UIインタラクションとナビゲーションをテスト
- フロントエンドとバックエンドの統合を検証
- 本番環境へのデプロイメントに向けて準備

## 仕組み

e2e-runnerエージェントは以下を実行します：

1. **ユーザーフローを分析**してテストシナリオを特定
2. **Playwrightテストを生成** - ページオブジェクトモデルパターンを使用
3. **複数のブラウザでテストを実行** (Chrome, Firefox, Safari)
4. **失敗をキャプチャ** - スクリーンショット、ビデオ、トレース付き
5. **レポートを生成** - 結果と成果物を含む
6. **不安定なテストを特定** - 修正方法を提案

## 使用例

```
ユーザー: /e2e 市場検索と表示フローをテストしてください

エージェント (e2e-runner):
# E2E テスト生成: 市場検索と表示フロー

## テストシナリオの特定

**ユーザージャーニー:** 市場検索 → 結果表示 → 市場をクリック → 詳細表示

**テストカバレッジ:**
1. 市場ページに移動
2. セマンティック検索を実行
3. 検索結果を確認
4. 最初の結果をクリック
5. 市場詳細ページが読み込まれることを確認
6. チャートがレンダリングされることを確認

## 生成されたテストコード

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('Market Search and View Flow', () => {
  test('user can search markets and view details', async ({ page }) => {
    // 1. 市場ページに移動
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // ページが読み込まれたことを確認
    await expect(page).toHaveTitle(/Markets/)
    await expect(page.locator('h1')).toContainText('Markets')

    // 2. セマンティック検索を実行
    await marketsPage.searchMarkets('election')

    // API レスポンスを待つ
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. 検索結果を確認
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // 検索結果のスクリーンショットを撮影
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. 最初の結果をクリック
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. 市場詳細ページが読み込まれたことを確認
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. チャートがレンダリングされたことを確認
    await expect(detailsPage.priceChart).toBeVisible()

    // 市場名が一致することを確認
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // 市場詳細のスクリーンショットを撮影
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('search with no results shows empty state', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 存在しない市場を検索
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // 空の状態を確認
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /no.*results|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('can clear search and see all markets again', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 初期市場数を記録
    const initialCount = await marketsPage.marketCards.count()

    // 検索を実行
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // フィルタリングされた結果を確認
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // 検索をクリア
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // すべての市場が再度表示されたことを確認
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
```

## テストの実行

```bash
# 生成されたテストを実行
npx playwright test tests/e2e/markets/search-and-view.spec.ts

3つのワーカーを使用して3つのテストを実行中

  ✓  [chromium] › search-and-view.spec.ts:5:3 › user can search markets and view details (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › search with no results shows empty state (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › can clear search and see all markets again (2.9s)

  3つが成功 (9.1s)

生成された成果物:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## テストレポート

```
╔══════════════════════════════════════════════════════════════╗
║                  E2E テスト結果                              ║
╠══════════════════════════════════════════════════════════════╣
║ ステータス:  ✅ すべてのテストが成功                          ║
║ 合計:        3つのテスト                                    ║
║ 成功:        3 (100%)                                      ║
║ 失敗:        0                                              ║
║ 不安定:      0                                              ║
║ 実行時間:    9.1s                                          ║
╚══════════════════════════════════════════════════════════════╝

成果物:
📸 スクリーンショット: 2ファイル
📹 ビデオ: 0ファイル (失敗時のみ)
🔍 トレース: 0ファイル (失敗時のみ)
📊 HTMLレポート: playwright-report/index.html

レポートを表示: npx playwright show-report
```

✅ E2Eテストスイートは CI/CD統合の準備ができています！
```

## テスト成果物

テストが実行されると、以下の成果物がキャプチャされます：

**すべてのテストで:**
- タイムラインと結果を含むHTMLレポート
- CI統合用のJUnit XML

**失敗時のみ:**
- 失敗時の状態のスクリーンショット
- テストのビデオ記録
- デバッグ用のトレースファイル（ステップバイステップ再生）
- ネットワークログ
- コンソールログ

## 成果物の表示

```bash
# HTMLレポートをブラウザで表示
npx playwright show-report

# 特定のトレースファイルを表示
npx playwright show-trace artifacts/trace-abc123.zip

# スクリーンショットは artifacts/ ディレクトリに保存されます
open artifacts/search-results.png
```

## 不安定なテスト検出

テストが間欠的に失敗する場合:

```
⚠️  不安定なテストが検出されました: tests/e2e/markets/trade.spec.ts

テストは10回の実行中7回成功 (成功率 70%)

一般的な失敗:
"要素 '[data-testid="confirm-btn"]' の待機がタイムアウト"

推奨される修正:
1. 明示的な待機を追加: await page.waitForSelector('[data-testid="confirm-btn"]')
2. タイムアウトを増加: { timeout: 10000 }
3. コンポーネント内の競合状態をチェック
4. 要素がアニメーションで隠れていないか確認

隔離の推奨: 修正されるまで test.fixme() としてマーク
```

## ブラウザ設定

テストはデフォルトで複数のブラウザで実行されます:
- ✅ Chromium (Desktop Chrome)
- ✅ Firefox (Desktop)
- ✅ WebKit (Desktop Safari)
- ✅ Mobile Chrome (オプション)

`playwright.config.ts` で設定を調整してブラウザを変更できます。

## CI/CD 統合

CI パイプラインに以下を追加します:

```yaml
# .github/workflows/e2e.yml
- name: Playwrightをインストール
  run: npx playwright install --with-deps

- name: E2Eテストを実行
  run: npx playwright test

- name: 成果物をアップロード
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX固有の重要なフロー

PMXについては、以下のE2Eテストを優先します:

**🔴 重要 (常に成功する必要がある):**
1. ユーザーはウォレットを接続できる
2. ユーザーは市場を参照できる
3. ユーザーは市場を検索できる (セマンティック検索)
4. ユーザーは市場詳細を表示できる
5. ユーザーはトレードを配置できる (テスト資金を使用)
6. 市場は正しく解決される
7. ユーザーは資金を引き出すことができる

**🟡 重要:**
1. 市場作成フロー
2. ユーザープロフィール更新
3. リアルタイム価格更新
4. チャートレンダリング
5. 市場のフィルタリングとソート
6. モバイルレスポンシブレイアウト

## ベストプラクティス

**推奨事項:**
- ✅ 保守性のためにページオブジェクトモデルを使用
- ✅ セレクター用に data-testid 属性を使用
- ✅ 任意のタイムアウトではなくAPIレスポンスを待機
- ✅ 重要なユーザージャーニーをエンドツーエンドでテスト
- ✅ mainにマージする前にテストを実行
- ✅ テストが失敗したときに成果物を確認

**推奨されない事項:**
- ❌ 脆弱なセレクター (CSSクラスは変更される可能性があります) を使用
- ❌ 実装の詳細をテスト
- ❌ 本番環境に対してテストを実行
- ❌ 不安定なテストを無視
- ❌ 失敗時の成果物の確認をスキップ
- ❌ E2Eですべてのエッジケースをテスト (単体テストを使用してください)

## 重要な注記

**PMX向けの重要事項:**
- 実際のお金に関わるE2Eテストはテストネット/ステージングのみで実行する必要があります
- トレーディングテストを本番環境に対して実行しないでください
- 金融テスト用に `test.skip(process.env.NODE_ENV === 'production')` を設定してください
- 小さなテスト資金を持つテストウォレットのみを使用してください

## 他のコマンドとの統合

- `/plan` を使用して、テスト対象の重要なジャーニーを特定
- `/tdd` を使用して単体テスト (より高速で詳細度が高い)
- `/e2e` を使用して統合とユーザージャーニーテスト
- `/code-review` を使用してテスト品質を確認

## 関連エージェント

このコマンドは以下の場所にある `e2e-runner` エージェントを呼び出します:
`~/.claude/agents/e2e-runner.md`

## クイックコマンド

```bash
# すべてのE2Eテストを実行
npx playwright test

# 特定のテストファイルを実行
npx playwright test tests/e2e/markets/search.spec.ts

# ヘッドレスモードで実行 (ブラウザを表示)
npx playwright test --headed

# テストをデバッグ
npx playwright test --debug

# テストコードを生成
npx playwright codegen http://localhost:3000

# レポートを表示
npx playwright show-report
```
