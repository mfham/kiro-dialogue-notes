# サブエージェント連携スキル

## 役割
メインエージェントから呼び出されるサブエージェントとして動作する。

## ユースケース

### 1. 開発フロー全体の自動化
**シナリオ:** コミット前チェック → コミット → 学びの記録

```
「privacy-guardでファイルをチェックして、問題なければコミットして、学びを記録して」
```

**フロー:**
1. privacy-guardサブエージェントで個人情報チェック
2. 問題なければgit commit実行
3. learning-trackerサブエージェントで学びを記録

### 2. コードレビュー後の学び記録
**シナリオ:** レビューコメントを分析して自動で学びを抽出

```
「このPRのレビューコメントから学びを抽出して、learning-trackerに記録して」
```

**フロー:**
1. メインエージェントがレビューコメントを分析
2. 学びポイントを抽出
3. learning-trackerサブエージェントに渡して記録

### 3. 定期的な振り返り
**シナリオ:** 週末に自動で週次レポートを生成

```
「今週の学びをlearning-trackerでまとめて、Slackに投稿して」
```

**フロー:**
1. learning-trackerサブエージェントで週次レポート生成
2. メインエージェントがSlack APIで投稿

### 4. 複数リポジトリの学び集約
**シナリオ:** 複数プロジェクトの学びを横断検索

```
「全プロジェクトのSecurityカテゴリの学びをlearning-trackerで検索して」
```

**フロー:**
1. learning-trackerサブエージェントで`~/.kiro/learnings/`を検索
2. Securityカテゴリの学びを抽出
3. メインエージェントが整形して表示

## サブエージェント呼び出し時の注意点

### 1. コンテキストの引き継ぎ
メインエージェントからサブエージェントに情報を渡す際は、明示的に指定：
- 例: 「リポジトリ名: my-project、カテゴリ: Security で記録して」

### 2. 結果の受け取り
サブエージェントが保存したファイルパスを受け取り、メインエージェントで後続処理に活用。

### 3. エラーハンドリング
サブエージェントがエラーを返した場合の処理を考慮：
- 例: 保存失敗時はリトライまたはユーザーに通知

## 連携パターン

### パターン1: 順次実行
```
privacy-guard → git commit → learning-tracker
```

### パターン2: 並行実行
```
[privacy-guard, learning-tracker(先月のレポート)] を同時実行
```

### パターン3: 条件分岐
```
if privacy-guard.has_issues:
  learning-tracker.record("個人情報検出の学び")
else:
  git commit
```

## 統合ワークフロー例

**「安全なコミットフロー」**
```
1. privacy-guardでスキャン
2. 問題があれば → learning-trackerに記録して終了
3. 問題なければ → git commit
4. コミット成功 → learning-trackerで学びを記録（オプション）
```

## Git操作の自動検出

### 方法1: シェル履歴の監視
```bash
LAST_CMD=$(history | tail -1)

if echo "$LAST_CMD" | grep -q "git commit"; then
  echo "💡 コミットしましたね！学びはありましたか？"
fi
```

### 方法2: Git hooksの活用
`~/.kiro/hooks/post-commit` を作成し、コミット後に自動実行：

```bash
#!/bin/bash
echo "💡 コミットお疲れ様です！"
echo "今回のコミットで学びはありましたか？ (y/n)"
read -r response

if [[ "$response" =~ ^[Yy]$ ]]; then
  kiro-cli chat --agent learning-tracker --message "今回のコミットで学んだことを記録したい"
fi
```

プロジェクトの `.git/hooks/post-commit` にシンボリックリンクを作成：
```bash
ln -s ~/.kiro/hooks/post-commit .git/hooks/post-commit
chmod +x ~/.kiro/hooks/post-commit
```

## 注意事項
- 自動検出は**オプション機能**として実装
- ユーザーが煩わしく感じないよう、頻度を調整可能に
- 「今日はもう聞かない」オプションを提供
- 設定ファイルで有効/無効を切り替え可能に
