# Kiro CLI 使い方ガイド

**公式ドキュメント**: https://kiro.dev/docs/

---

## 基本コマンド

### 起動・終了
```bash
kiro-cli chat          # チャット開始
/quit                  # 終了
kiro-cli --help        # ヘルプ表示
```

### モデル管理
```bash
/model                 # 現在のモデルと利用可能なモデル一覧を表示
```

## コア機能

### 1. コードインテリジェンス（LSP統合）

プロジェクトルートで初期化：
```bash
/code init
```

これにより `.kiro/settings/lsp.json` が作成され、言語サーバーが起動します。

**対応言語**
- TypeScript
- Rust
- Python
- Go
- Java
- Ruby
- C/C++

**主な機能**
- シンボル検索（関数、クラス、変数）
- 定義へのジャンプ
- 参照の検索
- コンパイラ診断・エラー表示
- 安全なシンボルリネーム

**無効化**
```bash
rm .kiro/settings/lsp.json
```

### 2. サブエージェント

複雑なタスクを独立したサブエージェントに委譲できます。

**使用例**
- 複数の独立したタスクを並列処理
- メインの会話コンテキストを汚さずに補助タスクを実行
- 最大4つのサブエージェントを同時実行可能

### 3. プランナーエージェント

**起動方法**
- `Shift + Tab` でトグル
- `/plan` コマンド
- `/plan <プロンプト>` で即座に開始

**特徴**
- 読み取り専用（ファイル変更なし）
- アイデアを構造化された実装計画に変換
- 要件収集とタスク分解に特化

**ワークフロー**
1. **要件収集** - 構造化された質問で要件を明確化
2. **調査・分析** - コードベース探索、技術調査
3. **実装計画** - ステップバイステップの詳細計画作成
4. **承認と引き継ぎ** - 計画承認後、実行エージェントに移行

**読み取り専用の範囲**
- ✓ ファイル読み取り、コードインテリジェンス、検索、Web検索
- ✗ ファイル書き込み、コマンド実行、MCPツール

**終了方法**
- `Shift + Tab` で前のエージェントに戻る
- 計画承認時に `y` で自動的に実行モードへ

## `.kiro/` ディレクトリ構成

### 1. steering/ - 動作ルール・ガイドライン

**場所**
- グローバル: `~/.kiro/steering/`
- ワークスペース: `.kiro/steering/`

**機能**
- Kiroの動作をカスタマイズ
- コーディング規約、API設計、テスト戦略など
- AGENTS.md標準もサポート

**インクルージョンモード**

常に含める（デフォルト）:
```yaml
---
inclusion: always
---
```

特定ファイルで自動含める:
```yaml
---
inclusion: fileMatch
fileMatchPattern: "components/**/*.tsx"
---
```

手動で呼び出し（`#steering-file-name`）:
```yaml
---
inclusion: manual
---
```

説明に基づいて自動判定:
```yaml
---
inclusion: auto
name: api-design
description: REST API design patterns
---
```

**ファイル参照**
```markdown
#[[file:api/openapi.yaml]]
```

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

---

## `.kiro/` ディレクトリ構成（続き）

### 2. settings/lsp.json - LSP設定

**場所**: `.kiro/settings/lsp.json` (プロジェクトルート)

**作成**: `/code init`

**機能**: 言語サーバーの設定

### 3. MCP設定 - Model Context Protocol

**設定方法**
1. MCP設定ファイルを作成
2. `Cmd + ,` (Mac) / `Ctrl + ,` (Windows/Linux) で設定を開く
3. "MCP"を検索して有効化

**機能**
- 外部ツールやAPIとの統合
- 専門知識ベースへのアクセス
- カスタムツールの追加

**トラブルシューティング**
- Kiroパネル → Output → "Kiro - MCP Logs"

## 利用可能なツール

### ファイル操作
- `fs_read` - ファイル・ディレクトリ読み取り
- `fs_write` - ファイル作成・編集
- `glob` - パターンマッチでファイル検索
- `grep` - テキスト検索

### コード操作
- `code` - コードインテリジェンス（シンボル検索、定義ジャンプなど）

### 実行
- `execute_bash` - Bashコマンド実行
- `use_aws` - AWS CLI呼び出し

### Web
- `web_search` - Web検索
- `web_fetch` - URL取得

### その他
- `use_subagent` - サブエージェント起動

## ベストプラクティス

### ステアリングファイル
- **焦点を絞る**: 1ファイル1ドメイン
- **明確な命名**: `api-rest-conventions.md`, `testing-unit-patterns.md`
- **理由を含める**: 決定の背景を説明
- **例を提供**: コードスニペット、before/after
- **セキュリティ優先**: APIキーや機密情報は含めない
- **定期的にメンテナンス**: スプリント計画時にレビュー

### 一般的なステアリングファイル例
- `api-standards.md` - REST規約、エラーレスポンス
- `testing-standards.md` - テストパターン、カバレッジ
- `code-conventions.md` - 命名規則、ファイル構成
- `security-policies.md` - 認証、バリデーション
- `deployment-workflow.md` - ビルド、デプロイ手順

### コード作業
1. プロジェクトルートで `/code init` を実行
2. シンボル検索や定義ジャンプを活用
3. 複雑なタスクはサブエージェントに委譲

### 計画立案
1. `Shift + Tab` でプランナーを起動
2. 要件を整理してタスク分解
3. メインエージェントで実装

### カスタマイズ
1. `~/.kiro/steering/` にガイドラインを作成
2. マークダウン形式で記述
3. ドメイン固有の知識やルールを定義

## トラブルシューティング

### LSPが動作しない
```bash
# 設定ファイルを削除して再初期化
rm .kiro/settings/lsp.json
/code init
```

### コンテキストが肥大化
- サブエージェントを活用して独立したタスクを分離
- 新しいチャットセッションを開始

## 参考リンク

- AWS料金計算: https://calculator.aws
- Kiro CLI: `kiro-cli --help`
