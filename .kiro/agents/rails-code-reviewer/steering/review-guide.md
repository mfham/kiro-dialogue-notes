# Railsコードレビューガイド

## DHHの思想・Rails Way

### The Rails Doctrine（Rails の教義）

**Convention over Configuration（設定より規約）**
- Railsの規約に従う（ファイル配置、命名規則）
- 設定ファイルを増やさない
- フレームワークの流儀に逆らわない

**Omakase（おまかせ）**
- Railsが提供するフルスタックを活用
- 過度な抽象化を避ける
- 「シンプルさ」を優先

**Majestic Monolith（壮大なモノリス）**
- マイクロサービスより最初はモノリス
- 適切に構造化されたモノリスの価値
- 早すぎる分割を避ける

**Progress over Stability（安定性より進歩）**
- 新しい機能を積極的に採用
- レガシーに縛られない
- 継続的な改善

**Optimize for Programmer Happiness（プログラマーの幸福を最適化）**
- 読みやすく、書きやすいコード
- 過度な抽象化より明確さ
- テストは重要だが、100%カバレッジに固執しない

**Provide Sharp Knives（鋭いナイフを提供）**
- 強力なツールを信頼して使う
- 過保護な制約を避ける
- 開発者を信頼する

## レビュー優先度

### 🔴 Critical（必須修正）
- セキュリティ脆弱性
- SQLインジェクション、XSS
- 認証・認可の欠落
- 本番コードのN+1クエリ
- Rails規約の重大な違反

### 🟡 Important（修正推奨）
- パフォーマンス問題
- Fat Controller
- テストカバレッジ不足（重要な部分のみ）
- エラーハンドリング不備
- アクセシビリティ問題

### 🟢 Nice to Have（検討推奨）
- コードスタイル改善
- リファクタリング機会
- ドキュメント追加

## DHH流 Rubyコード

### 明確さ > 簡潔さ
```ruby
# 過度に簡潔（避ける）
u&.p&.n || 'N/A'

# 明確
user&.profile&.name || 'N/A'
```

### 適切なRubyイディオム
```ruby
# Bad
array = []
items.each { |item| array << item.name }

# Good
items.map(&:name)

# Bad
if !user.admin?

# Good
unless user.admin?
```

## Rails Way

### コントローラー
```ruby
# DHH流 - シンプルで明確
class ArticlesController < ApplicationController
  def create
    @article = Article.new(article_params)
    
    if @article.save
      redirect_to @article
    else
      render :new, status: :unprocessable_entity
    end
  end
end
```

**避けるべき過度な抽象化:**
- 不要なサービスオブジェクト（単純なCRUDには不要）
- 過度なデザインパターン適用
- 1行のためのヘルパーメソッド

### モデル
```ruby
# Good - Active Recordを信頼
class Article < ApplicationRecord
  belongs_to :user
  has_many :comments, dependent: :destroy
  
  validates :title, presence: true
  
  scope :published, -> { where.not(published_at: nil) }
  
  def publish!
    update!(published_at: Time.current)
  end
end
```

**Active Recordを活用:**
- コールバックは適切に使う（過度に避けない）
- scopeで可読性を高める
- 関連を明確に定義

### クエリ最適化
```ruby
# Bad - N+1
@posts = Post.all
@posts.each { |post| post.author.name }

# Good
@posts = Post.includes(:author)

# Better - 必要なカラムのみ
@posts = Post.includes(:author).select(:id, :title, :author_id)
```

## セキュリティ（Rails Way）

### Strong Parameters
```ruby
# Good - 明確で保守しやすい
def article_params
  params.require(:article).permit(:title, :body, :published_at)
end
```

### 認証・認可
- Deviseなど実績あるgemを使う
- 車輪の再発明をしない
- シンプルな実装を優先

## フロントエンド（Hotwire優先）

### DHHの推奨スタック
- **Hotwire（Turbo + Stimulus）** - SPAより優先
- **Tailwind CSS** - ユーティリティファースト
- **Import Maps** - ビルドステップを最小化

### Stimulus
```javascript
// Good - シンプルで明確
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["output"]
  
  connect() {
    console.log("Hello, Stimulus!")
  }
  
  greet() {
    this.outputTarget.textContent = "Hello!"
  }
}
```

**原則:**
- JavaScriptは最小限に
- サーバーサイドレンダリング優先
- 必要な箇所のみインタラクティブに

## テスト（DHH流）

### 実用的なテスト
```ruby
# Good - 重要な動作をテスト
test "should create article" do
  assert_difference("Article.count") do
    post articles_url, params: { article: { title: "Test" } }
  end
  
  assert_redirected_to article_url(Article.last)
end
```

**テスト哲学:**
- 100%カバレッジに固執しない
- 重要な動作をテスト
- 実装の詳細ではなく動作をテスト
- システムテストで主要フローを確認

### 避けるべきテスト
- 過度なモック・スタブ
- 実装の詳細のテスト
- 脆弱なテスト（時間依存、順序依存）

## アンチパターン

### 過度な抽象化
```ruby
# Bad - 不要な抽象化
class ArticleCreationService
  def initialize(params)
    @params = params
  end
  
  def call
    Article.create(@params)
  end
end

# Good - シンプルに
@article = Article.create(article_params)
```

### 過度なDRY
- 重複を恐れすぎない
- 明確さを優先
- 早すぎる抽象化を避ける

### マイクロサービス症候群
- 最初はモノリスで
- 本当に必要になるまで分割しない
- Rails Enginesで十分な場合も

## よくある問題と解決

### Fat Controller
```ruby
# 複雑なビジネスロジックがある場合のみサービスオブジェクト
class ArticlePublisher
  def initialize(article)
    @article = article
  end
  
  def publish
    @article.transaction do
      @article.publish!
      NotificationMailer.published(@article).deliver_later
      @article.author.increment_published_count!
    end
  end
end

# 単純なCRUDならコントローラーで十分
```

### インデックス不足
- 外部キーには必ずインデックス
- WHERE句で使用するカラムにインデックス
- `rails db:migrate`後に`EXPLAIN`で確認

## レビュー出力フォーマット

各問題について:
1. **重要度**: 🔴 Critical / 🟡 Important / 🟢 Nice to Have
2. **場所**: ファイルパスと行番号
3. **問題**: 明確な説明
4. **推奨事項**: DHH流の解決方法（コード例付き）
5. **理由**: Rails Wayの観点から説明

## 参考リソース
- The Rails Doctrine: https://rubyonrails.org/doctrine
- DHH's blog: https://world.hey.com/dhh
- Rails guides: https://guides.rubyonrails.org/
