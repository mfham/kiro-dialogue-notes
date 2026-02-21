# 検索・一覧表示スキル

## 役割
保存された学びを検索・フィルタリングして表示する。

## 検索方法

### 1. 全件一覧表示
```bash
ls -lt ~/.kiro/learnings/*.md | head -20
```

### 2. 日付範囲で検索
```bash
# 今月の学び
CURRENT_MONTH=$(date +%Y-%m)
ls ~/.kiro/learnings/${CURRENT_MONTH}-*.md

# 先月の学び
LAST_MONTH=$(date -v-1m +%Y-%m)
ls ~/.kiro/learnings/${LAST_MONTH}-*.md

# 特定期間（例: 2026年1月）
ls ~/.kiro/learnings/2026-01-*.md
```

### 3. カテゴリで検索
```bash
# Securityカテゴリの学び
grep -l "category:.*Security" ~/.kiro/learnings/*.md
```

### 4. タグで検索
```bash
# gitタグの学び
grep -l "tags:.*git" ~/.kiro/learnings/*.md
```

### 5. キーワード検索
```bash
# 本文中のキーワード検索
grep -l "パフォーマンス" ~/.kiro/learnings/*.md
```

## 表示フォーマット

```markdown
## 検索結果: <条件>

### 2026-02-21 | Security, Code Quality
**タイトル:** 入力値バリデーションの重要性
**タグ:** git, review, validation
**ファイル:** ~/.kiro/learnings/2026-02-21-a1b2c3d4.md

---

### 2026-02-20 | Performance
**タイトル:** クエリ最適化でレスポンス改善
**タグ:** database, sql, optimization
**ファイル:** ~/.kiro/learnings/2026-02-20-b2c3d4e5.md

---

**合計:** 2件の学びが見つかりました
```

## 詳細表示

特定の学びの詳細を表示する場合：

```bash
cat ~/.kiro/learnings/2026-02-21-a1b2c3d4.md
```

## 使用例

- 「先月の学びを見せて」
- 「Securityカテゴリの学びを表示」
- 「gitタグの学びを検索」
- 「パフォーマンスに関する学びを検索」
