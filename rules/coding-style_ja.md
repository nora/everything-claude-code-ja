# コーディングスタイル

## イミュータビリティ（不変性）（重要）

常に新しいオブジェクトを作成し、決してミューテーション（変更）をしないこと：

```javascript
// 誤り：ミューテーション
function updateUser(user, name) {
  user.name = name  // ミューテーション！
  return user
}

// 正解：イミュータビリティ（不変性）
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## ファイル構成

少数の大きなファイルより多くの小さなファイル：
- 高い凝聚度、低い結合度
- 標準は200～400行、最大800行
- 大きなコンポーネントからユーティリティを抽出する
- タイプ別ではなく、機能/ドメイン別に整理する

## エラーハンドリング

常に包括的にエラーをハンドリング：

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('操作が失敗しました:', error)
  throw new Error('詳細でユーザーフレンドリーなメッセージ')
}
```

## 入力値の検証

常にユーザー入力を検証：

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## コード品質チェックリスト

作業の完了とマークする前に：
- [ ] コードは可読性があり、適切に名前が付けられている
- [ ] 関数は小さい（50行未満）
- [ ] ファイルはフォーカスされている（800行未満）
- [ ] ネストが深くない（4段階以下）
- [ ] 適切なエラーハンドリング
- [ ] console.logステートメントがない
- [ ] ハードコードされた値がない
- [ ] ミューテーションがない（イミュータブルなパターンを使用）
