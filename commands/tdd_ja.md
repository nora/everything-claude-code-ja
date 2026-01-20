---
description: テスト駆動開発ワークフローを実行。インターフェースをスキャフォルドし、テストを最初に生成してから、最小限のコードを実装する。カバレッジは80%以上を確保。
---

# TDDコマンド

このコマンドは**tdd-guide**エージェントを呼び出して、テスト駆動開発の方法論を実装します。

## このコマンドが実行すること

1. **インターフェースのスキャフォルド** - 型/インターフェースを最初に定義
2. **テストを最初に生成** - 失敗するテストを作成（RED）
3. **最小限のコード実装** - 合格するのに必要最小限のコードを作成（GREEN）
4. **リファクタリング** - テストを緑の状態に保ちながらコードを改善（REFACTOR）
5. **カバレッジを確認** - 80%以上のテストカバレッジを確保

## 使用する場合

`/tdd` を使用する場合：
- 新機能を実装する
- 新しい関数/コンポーネントを追加する
- バグを修正する（バグを再現するテストを最初に作成）
- 既存コードをリファクタリングする
- クリティカルなビジネスロジックを構築する

## 動作方法

tdd-guideエージェントは以下を実行します：

1. **インターフェースを定義** - 入力/出力用のインターフェース
2. **失敗するテストを作成** （コードがまだ存在しないため）
3. **テストを実行** - 失敗することを確認し、正しい理由で失敗していることを検証
4. **最小限の実装を作成** - テストに合格するのに必要最小限の実装
5. **テストを実行** - 合格することを検証
6. **コードをリファクタリング** - テストを緑の状態に保ちながらコードを改善
7. **カバレッジをチェック** - 80%以下の場合はテストを追加

## TDDサイクル

```
RED → GREEN → REFACTOR → 繰り返す

RED:      失敗するテストを作成
GREEN:    合格するのに必要最小限のコードを作成
REFACTOR: コードを改善し、テストを合格させ続ける
REPEAT:   次の機能/シナリオへ
```

## 使用例

```
ユーザー: /tdd 市場の流動性スコアを計算する関数が必要です

エージェント (tdd-guide):
# TDDセッション：市場流動性スコア計算機

## ステップ1：インターフェースの定義（スキャフォルド）

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: 実装
  throw new Error('Not implemented')
}
```

## ステップ2：失敗するテストを作成（RED）

```typescript
// lib/liquidity.test.ts
import { calculateLiquidityScore } from './liquidity'

describe('calculateLiquidityScore', () => {
  it('should return high score for liquid market', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('should return low score for illiquid market', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000) // 1 day ago
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeLessThan(30)
    expect(score).toBeGreaterThanOrEqual(0)
  })

  it('should handle edge case: zero volume', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBe(0)
  })
})
```

## ステップ3：テストを実行 - 失敗を確認

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ should return high score for liquid market (2 ms)
    Error: Not implemented

1 test failed, 0 passed
```

✅ テストは予想通り失敗しました。実装の準備ができています。

## ステップ4：最小限のコードを実装（GREEN）

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // ゼロボリュームのエッジケースを処理
  if (market.totalVolume === 0) {
    return 0
  }

  // コンポーネントスコアを計算（0-100スケール）
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // 最近のアクティビティボーナス
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // 加重平均
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // 0-100にクランプ
}
```

## ステップ5：テストを実行 - 合格を確認

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

✅ すべてのテストが合格しました！

## ステップ6：リファクタリング（改善）

```typescript
// lib/liquidity.ts - 定数と読みやすさを改善したリファクタリング版
const WEIGHTS = {
  VOLUME: 0.4,
  SPREAD: 0.3,
  TRADERS: 0.2,
  RECENCY: 0.1,
} as const

const SCALE_FACTORS = {
  VOLUME: 1000,
  SPREAD: 1000,
  TRADERS: 10,
  RECENCY_PENALTY: 10,
} as const

function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}

export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0

  const volumeScore = Math.min(market.totalVolume / SCALE_FACTORS.VOLUME, 100)
  const spreadScore = clamp(100 - (market.bidAskSpread * SCALE_FACTORS.SPREAD), 0, 100)
  const traderScore = Math.min(market.activeTraders / SCALE_FACTORS.TRADERS, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = clamp(100 - (hoursSinceLastTrade * SCALE_FACTORS.RECENCY_PENALTY), 0, 100)

  const weightedScore =
    volumeScore * WEIGHTS.VOLUME +
    spreadScore * WEIGHTS.SPREAD +
    traderScore * WEIGHTS.TRADERS +
    recencyScore * WEIGHTS.RECENCY

  return clamp(weightedScore, 0, 100)
}
```

## ステップ7：テストが引き続き合格することを確認

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

✅ リファクタリング完了、テストは引き続き合格しています！

## ステップ8：カバレッジをチェック

```bash
npm test -- --coverage lib/liquidity.test.ts

File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

Coverage: 100% ✅ （目標：80%）
```

✅ TDDセッション完了！
```

## TDDのベストプラクティス

**実施すること：**
- ✅ 実装の前にまずテストを作成
- ✅ テストを実行して失敗することを確認してから実装
- ✅ テストに合格するのに必要最小限のコードを作成
- ✅ リファクタリングはテストが緑の状態になった後のみ
- ✅ エッジケースとエラーシナリオを追加
- ✅ 80%以上のカバレッジを目指す（クリティカルコードは100%）

**実施しないこと：**
- ❌ テストの前に実装を作成
- ❌ 各変更後のテスト実行をスキップ
- ❌ 一度に書きすぎ
- ❌ 失敗するテストを無視
- ❌ 実装の詳細をテスト（動作をテスト）
- ❌ すべてをモック化（統合テストを優先）

## 含めるテストの種類

**ユニットテスト**（関数レベル）：
- 正常なシナリオ
- エッジケース（空、null、最大値）
- エラー条件
- 境界値

**統合テスト**（コンポーネントレベル）：
- APIエンドポイント
- データベース操作
- 外部サービス呼び出し
- React hooks付きコンポーネント

**E2Eテスト**（`/e2e`コマンドを使用）：
- クリティカルなユーザーフロー
- 複数ステップのプロセス
- フルスタック統合

## カバレッジ要件

- **最小80%** すべてのコード
- **100%が必須** ：
  - 財務計算
  - 認証ロジック
  - セキュリティ関連コード
  - コアビジネスロジック

## 重要な注意事項

**必須**：テストは実装の前に作成する必要があります。TDDサイクルは以下の通りです：

1. **RED** - 失敗するテストを作成
2. **GREEN** - 合格するように実装
3. **REFACTOR** - コードを改善

RED フェーズをスキップしないでください。テストの前にコードを書かないでください。

## 他のコマンドとの統合

- まず`/plan`を使用して何を構築するかを理解
- `/tdd`を使用してテストで実装
- ビルドエラーが発生した場合は`/build-and-fix`を使用
- 実装をレビューするには`/code-review`を使用
- カバレッジを確認するには`/test-coverage`を使用

## 関連エージェント

このコマンドは`tdd-guide`エージェントを呼び出します。ロケーション：
`~/.claude/agents/tdd-guide.md`

および`tdd-workflow`スキルを参照できます。ロケーション：
`~/.claude/skills/tdd-workflow/`
