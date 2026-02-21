# 学びの記録スキル

## 役割
ユーザーから学びの内容をヒアリングし、構造化して`~/.kiro/learnings/`に保存する。

## 記録フロー

### ステップ1: ヒアリング
以下の3つの質問で情報を収集：

1. **「どんな状況でしたか？」**
   - コミット時、レビュー時、実装時、デバッグ時など
   - 簡潔に1-2文で

2. **「具体的にどんな気づきや学びがありましたか？」**
   - 技術的な発見、ベストプラクティス、失敗から学んだこと
   - できるだけ具体的に

3. **「今後どうしますか？何を意識しますか？」**
   - 次回から気をつけること、試してみたいこと
   - アクションアイテム

### ステップ2: カテゴリとタグの提案
ユーザーの回答を分析し、適切なカテゴリとタグを提案。

**カテゴリ例:**
- Security（セキュリティ）
- Performance（パフォーマンス）
- Code Quality（コード品質）
- Git（Git操作）
- Review（コードレビュー）
- Architecture（アーキテクチャ）
- Testing（テスト）
- Debugging（デバッグ）
- Refactoring（リファクタリング）
- Documentation（ドキュメント）
- Tooling（ツール）
- Best Practices（ベストプラクティス）

**カテゴリ提案ロジック:**
- キーワードマッチング（例: "脆弱性", "認証" → Security）
- 複数カテゴリも可能（例: Security + Code Quality）
- ユーザーに確認して調整可能

**タグ提案ロジック:**
- 技術キーワードを抽出（例: git, docker, typescript, react）
- アクションキーワード（例: review, refactoring, testing）
- 最大5個程度

### ステップ3: 保存処理

**保存スクリプト:**
```bash
# ディレクトリ作成
mkdir -p ~/.kiro/learnings

# UUID生成（短縮版: 8文字）
UUID=$(uuidgen | tr '[:upper:]' '[:lower:]' | cut -d'-' -f1)

# 日付取得
DATE=$(date +%Y-%m-%d)
DATETIME=$(date -u +"%Y-%m-%dT%H:%M:%S%z")

# ファイル名
FILENAME="$DATE-$UUID.md"
FILEPATH="$HOME/.kiro/learnings/$FILENAME"

# ファイル作成
cat > "$FILEPATH" << 'LEARNING'
---
id: $UUID
date: $DATETIME
category: [Security, Performance]
tags: [git, review]
repository: my-project
---
# 学びのタイトル

## 状況
コミット時にレビュー指摘を受けた

## 気づき・学び
入力値のバリデーションが不十分だった

## アクション
今後は必ず入力値チェックを実装する
LEARNING

echo "✅ 学びを記録しました: $FILEPATH"
```

### ステップ4: 確認とフィードバック
- 保存したファイルパスを表示
- ポジティブなフィードバック（「いい学びですね！成長してますよ！」）
- 確認方法を案内（`cat ~/.kiro/learnings/<filename>`）

## 保存形式

```yaml
---
id: <uuid>
date: 2026-02-21T01:23:00+09:00
category: [Security, Performance]
tags: [git, review, refactoring]
repository: <repo-name or null>
---
# 学びのタイトル

## 状況
何をしていたか

## 気づき・学び
具体的な学び内容

## アクション
今後どうするか
```

## Git情報の活用

コミット直後の記録時は、Git情報を自動取得して参考にする：

```bash
# 最新のコミットメッセージ
git log -1 --pretty=%B

# 変更されたファイル
git diff --name-only HEAD~1

# 変更内容のサマリー
git diff --stat HEAD~1
```

これらを「状況」セクションに自動入力して効率化。
