# kiro-dialogue-notes

**公式ドキュメント**: https://kiro.dev/docs/

---

## ステアリングファイルとAGENTS.md標準

### ステアリングファイル

`~/.kiro/steering/` ディレクトリに配置するMarkdownファイルで、Kiroの動作をカスタマイズする指示書。会話の冒頭で自動的にコンテキストとして読み込まれる。

#### 基本的な使い方

**常に適用（デフォルト）**
```markdown
# 日本語使用ルール

- すべての回答は日本語で行うこと
- 技術用語は英語併記
```

**条件付き適用**
```markdown
---
inclusion: conditional
fileMatchPattern: "**/*.py"
---

# Python専用ルール

- 型ヒントを必ず使用
- docstringはGoogle形式
```

#### fileMatchPattern の例

- `"**/*.ts"` - TypeScriptファイルのみ
- `"**/*.{js,jsx}"` - JavaScriptとJSXファイル
- `"src/**/*.java"` - srcディレクトリ内のJavaファイル
- `"**/test/**"` - testディレクトリ内の全ファイル

---

### AGENTS.md標準

プロジェクトルートに配置する特別なファイル（`AGENTS.md` または `.github/AGENTS.md`）で、そのプロジェクト専用のAI動作ルールを定義する。

#### ステアリングファイルとの違い

| 項目 | ステアリング | AGENTS.md |
|------|-------------|-----------|
| 場所 | `~/.kiro/steering/` | プロジェクトルート |
| スコープ | 全プロジェクト共通 | そのプロジェクトのみ |
| 用途 | 個人の好み・汎用ルール | プロジェクト固有ルール |
| 共有 | 個人用 | チームで共有可能（Git管理） |

#### 使用例

```markdown
# Project AI Guidelines

## Architecture

- このプロジェクトはClean Architectureを採用
- `domain/` - ビジネスロジック
- `infrastructure/` - 外部依存
- `application/` - ユースケース

## Coding Standards

- テストは `__tests__/` ディレクトリに配置
- コンポーネントは必ずStorybookを作成
- API呼び出しは必ず `services/api/` 経由
```

#### 優先順位

1. **AGENTS.md** - 最優先（プロジェクト固有）
2. **ステアリングファイル** - 次点（個人設定）
3. **デフォルト動作** - 最後

#### サブエージェントへの適用

- **AGENTS.mdとステアリングファイルは、サブエージェントにも適用される**
- サブエージェントは独立したコンテキストで動作するが、プロジェクトルールは継承する
- SUBAGENTS.mdのような別ファイルは不要

---

## 実験的機能

Kiro CLIには開発中の実験的機能があり、`/experiment` コマンドまたは設定で有効化できる。

### 管理方法

**対話的に管理**
```
/experiment
```

**設定コマンドで管理**
```bash
# 全実験的機能の確認
kiro-cli settings list | grep -i enable

# 個別に有効化/無効化
kiro-cli settings chat.enableFeatureName true
kiro-cli settings chat.enableFeatureName false
```

### 利用可能な実験的機能

| 機能 | 設定名 | 説明 |
|------|--------|------|
| **Knowledge management** | `chat.enableKnowledge` | セッション間で永続的なコンテキスト保存・検索 |
| **Tangent mode** | `chat.enableTangentMode` | 会話のチェックポイント作成（`/tangent` または Ctrl+T） |
| **TODO lists** | `chat.enableTodoList` | Kiroが自動でTODO管理（`/todo`） |
| **Thinking tool** | `chat.enableThinking` | AI推論プロセスの可視化 |
| **Checkpointing** | `chat.enableCheckpoint` | ファイル変更のスナップショット（`/checkpoint`） |
| **Context usage indicator** | `chat.enableContextUsageIndicator` | コンテキスト使用率をプロンプトに表示 |
| **Delegate** | `chat.enableDelegate` | 非同期タスク実行 |

### コンテキスト使用率表示

```bash
kiro-cli settings chat.enableContextUsageIndicator true
```

有効化すると、プロンプトに使用率が表示される：
- `[agent-name] 6% >` - 緑（<50%）
- `[agent-name] 75% >` - 黄（50-89%）
- `[agent-name] 95% >` - 赤（90-100%）

### 使用状況の確認

**チャット中**
```
/usage
```

**ダッシュボード**
```bash
kiro-cli dashboard
```

**プロフィール情報**
```bash
kiro-cli user profile
kiro-cli whoami
```

### 注意事項

- 実験的機能は予告なく変更・削除される可能性がある
- 本番環境での使用は慎重に
- 問題があれば `kiro issue` コマンドでフィードバック

詳細: https://kiro.dev/docs/cli/experimental/