# everything-claude-code-ja

当リポジトリは [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) のドキュメント類を和訳したものです。
contribution は受け付けていません。

---

# Claude Code のすべて

**Anthropic ハカソンの優勝者による Claude Code 構成の完全なコレクション。**

このリポジトリには、Claude Code で毎日使用する実稼働対応のエージェント、スキル、フック、コマンド、ルール、MCP 構成が含まれています。これらの構成は、実際の製品を構築する 10 か月以上の集中的な使用を経て進化しました。

---

## 最初に完全なガイドをお読みください

**これらの構成に入る前に、X:** に関する完全なガイドをお読みください。

<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />

**[すべての Claude Code の短縮ガイド](https://x.com/affaanmustafa/status/2012378465664745795)**

ガイドでは次のように説明されています。

- 各構成タイプの機能とそれをいつ使用するか
- Claude Code のセットアップを構成する方法
- コンテキスト ウィンドウの管理 (パフォーマンスにとって重要)
- 並列ワークフローと高度なテクニック
- これらの構成の背後にある哲学

**このリポジトリは構成のみです。ヒント、コツ、その他の例は、X の記事とビデオにあります (この Readme が進化するにつれてリンクが追加されます)。**

---

## 中身は何ですか

```
everything-claude-code/
|-- agents/           # 委任用の専門サブエージェント
|   |-- planner.md           # 機能実装計画
|   |-- architect.md         # システム設計の意思決定
|   |-- tdd-guide.md         # テスト駆動開発
|   |-- code-reviewer.md     # 品質とセキュリティレビュー
|   |-- security-reviewer.md # 脆弱性分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2Eテスト
|   |-- refactor-cleaner.md  # デッドコードのクリーンアップ
|   |-- doc-updater.md       # ドキュメント同期
|
|-- skills/           # ワークフロー定義とドメイン知識
|   |-- coding-standards.md         # 言語のベストプラクティス
|   |-- backend-patterns.md         # API、データベース、キャッシュパターン
|   |-- frontend-patterns.md        # React、Next.jsパターン
|   |-- project-guidelines-example.md # プロジェクト固有スキルの例
|   |-- tdd-workflow/               # TDD手法
|   |-- security-review/            # セキュリティチェックリスト
|   |-- clickhouse-io.md            # ClickHouseアナリティクス
|
|-- commands/         # クイック実行用スラッシュコマンド
|   |-- tdd.md              # /tdd - テスト駆動開発
|   |-- plan.md             # /plan - 実装計画
|   |-- e2e.md              # /e2e - E2Eテスト生成
|   |-- code-review.md      # /code-review - 品質レビュー
|   |-- build-fix.md        # /build-fix - ビルドエラー修正
|   |-- refactor-clean.md   # /refactor-clean - デッドコード削除
|   |-- test-coverage.md    # /test-coverage - カバレッジ分析
|   |-- update-codemaps.md  # /update-codemaps - ドキュメント更新
|   |-- update-docs.md      # /update-docs - ドキュメント同期
|
|-- rules/            # 常に従うべきガイドライン
|   |-- security.md         # 必須セキュリティチェック
|   |-- coding-style.md     # 不変性、ファイル構成
|   |-- testing.md          # TDD、80%カバレッジ要件
|   |-- git-workflow.md     # コミット形式、PRプロセス
|   |-- agents.md           # サブエージェントへの委任タイミング
|   |-- performance.md      # モデル選択、コンテキスト管理
|   |-- patterns.md         # APIレスポンス形式、フック
|   |-- hooks.md            # フックのドキュメント
|
|-- hooks/            # トリガーベースの自動化
|   |-- hooks.json          # PreToolUse、PostToolUse、Stopフック
|
|-- mcp-configs/      # MCPサーバー設定
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railwayなど
|
|-- plugins/          # プラグインエコシステムのドキュメント
|   |-- README.md           # プラグイン、マーケットプレイス、スキルガイド
|
|-- examples/         # 設定例
    |-- CLAUDE.md           # プロジェクトレベル設定の例
    |-- user-CLAUDE.md      # ユーザーレベル設定の例 (~/.claude/CLAUDE.md)
    |-- statusline.json     # カスタムステータスライン設定
```

---

## クイックスタート

### 1. 必要なものをコピーします

```bash
# リポジトリをクローン
git clone https://github.com/affaan-m/everything-claude-code.git

# エージェントをClaudeの設定フォルダにコピー
cp everything-claude-code/agents/*.md ~/.claude/agents/

# ルールをコピー
cp everything-claude-code/rules/*.md ~/.claude/rules/

# コマンドをコピー
cp everything-claude-code/commands/*.md ~/.claude/commands/

# スキルをコピー
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

### 2. settings.json にフックを追加します。

フックを `hooks/hooks.json` から `~/.claude/settings.json` にコピーします。

### 3. MCP を構成する

必要な MCP サーバーを `mcp-configs/mcp-servers.json` から `~/.claude.json` にコピーします。

**重要:** `YOUR_*_HERE` プレースホルダーを実際の API キーに置き換えます。

### 4. ガイドを読む

真剣に、[ガイドを読んでください](https://x.com/affaanmustafa/status/2012378465664745795)。これらの構成は、コンテキストと合わせて 10 倍意味を持ちます。

---

## 主要な概念

### エージェント

サブエージェントは、限定された範囲で委任されたタスクを処理します。例：

```markdown
---
name: code-reviewer
description: コードの品質、セキュリティ、保守性をレビュー
tools: Read, Grep, Glob, Bash
model: opus
---

あなたはシニアコードレビュアーです...
```

### スキル

スキルは、コマンドまたはエージェントによって呼び出されるワークフロー定義です。

```markdown
# TDD ワークフロー

1. まずインターフェースを定義
2. 失敗するテストを書く（RED）
3. 最小限のコードを実装（GREEN）
4. リファクタリング（IMPROVE）
5. 80%以上のカバレッジを確認
```

### フック

フックはツール イベントで起動されます。例 - console.log について警告します。

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [
    {
      "type": "command",
      "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[フック] console.logを削除してください' >&2"
    }
  ]
}
```

### ルール

ルールは常に従うべきガイドラインです。モジュール化しておいてください。

```
~/.claude/rules/
  security.md      # ハードコードされたシークレット禁止
  coding-style.md  # 不変性、ファイル制限
  testing.md       # TDD、カバレッジ要件
```

---

## Contributing

受け付けていません。

---

## 背景

私は実験的なロールアウト以来、Claude Code を使用してきました。 2025 年 9 月に Anthropic x Forum Ventures ハッカソンで [zenith.chat](https://zenith.chat) を [@DRodriguezFX](https://x.com/DRodriguezFX) とともに構築し、完全に Claude Code を使用して優勝しました。

これらの構成は、複数の実稼働アプリケーションにわたって徹底的にテストされています。

---

## 重要な注意事項

### コンテキストウィンドウの管理

**重要:** すべての MCP を一度に有効にしないでください。有効なツールが多すぎると、200k のコンテキスト ウィンドウが 70k に縮小する可能性があります。

経験則:

- 20 ～ 30 個の MCP が構成されている
- プロジェクトごとに 10 未満を有効にします
- アクティブなツールが 80 個未満

未使用のものを無効にするには、プロジェクト構成で `disabledMcpServers` を使用します。

### カスタマイズ

これらの構成は私のワークフローで機能します。あなたがすべき：

1. 共感できるものから始める
2. スタックに合わせて変更する
3. 使わないものは取り除く
4. 独自のパターンを追加する

---

## リンク

- **完全ガイド:** [Claude Code のすべての短縮ガイド](https://x.com/affaanmustafa/status/2012378465664745795)
- **フォロー:** [@Affanmustafa](https://x.com/affanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## ライセンス

MIT - 自由に使用し、必要に応じて変更し、可能であれば貢献してください。
