# クロードコードのすべて

**Anthropic ハカソンの優勝者による Claude Code 構成の完全なコレクション。**

このリポジトリには、Claude Code で毎日使用する実稼働対応のエージェント、スキル、フック、コマンド、ルール、MCP 構成が含まれています。これらの構成は、実際の製品を構築する 10 か月以上の集中的な使用を経て進化しました。

---

## 最初に完全なガイドをお読みください

**これらの構成に入る前に、X:** に関する完全なガイドをお読みください。


<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />


**[すべてのクロード コードの短縮ガイド](https://x.com/affaanmustafa/status/2012378465664745795)**



ガイドでは次のように説明されています。
- 各構成タイプの機能とそれをいつ使用するか
- クロードコードのセットアップを構成する方法
- コンテキスト ウィンドウの管理 (パフォーマンスにとって重要)
- 並列ワークフローと高度なテクニック
- これらの構成の背後にある哲学

**このリポジトリは構成のみです。ヒント、コツ、その他の例は、X の記事とビデオにあります (この Readme が進化するにつれてリンクが追加されます)。**

---

## 中身は何ですか

```
everything-claude-code/
|-- agents/           # Specialized subagents for delegation
|   |-- planner.md           # Feature implementation planning
|   |-- architect.md         # System design decisions
|   |-- tdd-guide.md         # Test-driven development
|   |-- code-reviewer.md     # Quality and security review
|   |-- security-reviewer.md # Vulnerability analysis
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E testing
|   |-- refactor-cleaner.md  # Dead code cleanup
|   |-- doc-updater.md       # Documentation sync
|
|-- skills/           # Workflow definitions and domain knowledge
|   |-- coding-standards.md         # Language best practices
|   |-- backend-patterns.md         # API, database, caching patterns
|   |-- frontend-patterns.md        # React, Next.js patterns
|   |-- project-guidelines-example.md # Example project-specific skill
|   |-- tdd-workflow/               # TDD methodology
|   |-- security-review/            # Security checklist
|   |-- clickhouse-io.md            # ClickHouse analytics
|
|-- commands/         # Slash commands for quick execution
|   |-- tdd.md              # /tdd - Test-driven development
|   |-- plan.md             # /plan - Implementation planning
|   |-- e2e.md              # /e2e - E2E test generation
|   |-- code-review.md      # /code-review - Quality review
|   |-- build-fix.md        # /build-fix - Fix build errors
|   |-- refactor-clean.md   # /refactor-clean - Dead code removal
|   |-- test-coverage.md    # /test-coverage - Coverage analysis
|   |-- update-codemaps.md  # /update-codemaps - Refresh docs
|   |-- update-docs.md      # /update-docs - Sync documentation
|
|-- rules/            # Always-follow guidelines
|   |-- security.md         # Mandatory security checks
|   |-- coding-style.md     # Immutability, file organization
|   |-- testing.md          # TDD, 80% coverage requirement
|   |-- git-workflow.md     # Commit format, PR process
|   |-- agents.md           # When to delegate to subagents
|   |-- performance.md      # Model selection, context management
|   |-- patterns.md         # API response formats, hooks
|   |-- hooks.md            # Hook documentation
|
|-- hooks/            # Trigger-based automations
|   |-- hooks.json          # PreToolUse, PostToolUse, Stop hooks
|
|-- mcp-configs/      # MCP server configurations
|   |-- mcp-servers.json    # GitHub, Supabase, Vercel, Railway, etc.
|
|-- plugins/          # Plugin ecosystem documentation
|   |-- README.md           # Plugins, marketplaces, skills guide
|
|-- examples/         # Example configurations
    |-- CLAUDE.md           # Example project-level config
    |-- user-CLAUDE.md      # Example user-level config (~/.claude/CLAUDE.md)
    |-- statusline.json     # Custom status line config
```

---

## クイックスタート

### 1. 必要なものをコピーします

```bash
# Clone the repo
git clone https://github.com/affaan-m/everything-claude-code.git

# Copy agents to your Claude config
cp everything-claude-code/agents/*.md ~/.claude/agents/

# Copy rules
cp everything-claude-code/rules/*.md ~/.claude/rules/

# Copy commands
cp everything-claude-code/commands/*.md ~/.claude/commands/

# Copy skills
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
description: Reviews code for quality, security, and maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior code reviewer...
```

### スキル

スキルは、コマンドまたはエージェントによって呼び出されるワークフロー定義です。

```markdown
# TDD Workflow

1. Define interfaces first
2. Write failing tests (RED)
3. Implement minimal code (GREEN)
4. Refactor (IMPROVE)
5. Verify 80%+ coverage
```

### フック

フックはツール イベントで起動されます。例 - console.log について警告します。

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### ルール

ルールは常に従うべきガイドラインです。モジュール化しておいてください。

```
~/.claude/rules/
  security.md      # No hardcoded secrets
  coding-style.md  # Immutability, file limits
  testing.md       # TDD, coverage requirements
```

---

## 貢献する

**貢献は歓迎されており、奨励されています。**

このリポジトリはコミュニティ リソースであることを目的としています。お持ちの場合:
- 便利なエージェントまたはスキル
- 賢いフック
- より優れた MCP 構成
- ルールの改善

ぜひ貢献してください！ガイドラインについては、[CONTRIBUTING.md](CONTRIBUTING.md) を参照してください。

### 貢献のアイデア

- 言語固有のスキル (Python、Go、Rust パターン)
- フレームワーク固有の構成 (Django、Rails、Laravel)
- DevOps エージェント (Kubernetes、Terraform、AWS)
- テスト戦略 (さまざまなフレームワーク)
- ドメイン固有の知識 (ML、データ エンジニアリング、モバイル)

---

＃＃ 背景

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

- **完全ガイド:** [クロード コードのすべての短縮ガイド](https://x.com/affaanmustafa/status/2012378465664745795)
- **フォロー:** [@Affanmustafa](https://x.com/affanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## ライセンス

MIT - 自由に使用し、必要に応じて変更し、可能であれば貢献してください。

---

**役立つ場合は、このリポジトリにスターを付けてください。ガイドを読んでください。素晴らしいものを構築しましょう。**