# プロジェクト例 CLAUDE.md

これはプロジェクトレベルのCLAUDE.mdファイルの例です。プロジェクトルートに配置してください。

## プロジェクト概要

[プロジェクトの簡潔な説明 - 何をするのか、使用技術スタックなど]

## 重要なルール

### 1. コード構成

- 少数の大きなファイルより多くの小さなファイル
- 高い結束度、低い結合度
- 通常は200-400行、最大800行/ファイル
- タイプ別ではなく機能/ドメイン別に整理

### 2. コードスタイル

- コード、コメント、ドキュメントに絵文字を使用しない
- 不変性を常に優先 - オブジェクトまたは配列を変更しない
- 本番コードにconsole.logを使用しない
- try/catchを使用した適切なエラーハンドリング
- ZodまたはそれらしいもでのInput検証

### 3. テスト

- TDD: 最初にテストを書く
- 最低80%のカバレッジ
- ユーティリティのユニットテスト
- APIのインテグレーションテスト
- 重要フローのE2Eテスト

### 4. セキュリティ

- ハードコードされたシークレットなし
- 機密データは環境変数を使用
- すべてのユーザー入力を検証
- パラメータ化クエリのみ使用
- CSRF保護が有効

## ファイル構造

```
src/
|-- app/              # Next.js アプリルーター
|-- components/       # 再利用可能なUIコンポーネント
|-- hooks/            # カスタムReactフック
|-- lib/              # ユーティリティライブラリ
|-- types/            # TypeScript型定義
```

## 主要パターン

### APIレスポンス形式

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
}
```

### エラーハンドリング

```typescript
try {
  const result = await operation()
  return { success: true, data: result }
} catch (error) {
  console.error('Operation failed:', error)
  return { success: false, error: 'ユーザーフレンドリーなメッセージ' }
}
```

## 環境変数

```bash
# 必須
DATABASE_URL=
API_KEY=

# オプション
DEBUG=false
```

## 利用可能なコマンド

- `/tdd` - テスト駆動開発ワークフロー
- `/plan` - 実装計画を作成
- `/code-review` - コード品質をレビュー
- `/build-fix` - ビルドエラーを修正

## Gitワークフロー

- Conventional Commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- mainに直接コミットしない
- PRはレビューが必須
- マージ前にすべてのテストが合格する必要がある
