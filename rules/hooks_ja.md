# フックシステム

## フックの種類

- **PreToolUse**: ツール実行前（検証、パラメータ修正）
- **PostToolUse**: ツール実行後（自動フォーマット、チェック）
- **Stop**: セッション終了時（最終検証）

## 現在のフック（~/.claude/settings.json内）

### PreToolUse
- **tmuxリマインダー**: 長時間実行コマンド（npm、pnpm、yarn、cargoなど）にtmuxを提案
- **gitプッシュレビュー**: プッシュ前にZedでレビューを開く
- **ドック（ドキュメント）ブロッカー**: 不要な.md/.txtファイルの作成をブロック

### PostToolUse
- **PR作成**: PR URLとGitHub Actionsステータスをログに記録
- **Prettier**: 編集後JS/TSファイルを自動フォーマット
- **TypeScriptチェック**: .ts/.tsxファイル編集後にtscを実行
- **console.logの警告**: 編集したファイルのconsole.logについて警告

### Stop
- **console.log監査**: セッション終了前に変更されたすべてのファイルのconsole.logをチェック

## 自動承認パーミッション

注意して使用してください：
- 信頼できる、明確に定義された計画に対して有効化
- 探索的な作業では無効化
- dangerously-skip-permissionsフラグは絶対に使用しないこと
- 代わりに~/.claude.json内の`allowedTools`を設定

## TodoWriteのベストプラクティス

TodoWriteツールは以下の用途で使用：
- マルチステップタスクの進捗を追跡
- 指示の理解を検証
- リアルタイムの方向性調整を有効化
- 細粒度の実装ステップを表示

Todoリストから以下が明らかになる：
- ステップの順序が間違っている場合
- 欠落しているアイテム
- 不要な余分なアイテム
- 粒度が不適切な場合
- 要件を誤解している場合
