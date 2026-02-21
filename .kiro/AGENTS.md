# Project Agents

このプロジェクトで利用可能なカスタムエージェント

## privacy-guard

個人情報検出エージェント

**役割:** ドキュメントやコードファイルから個人情報を検出し、GitHubへのプッシュ前に報告する

**使用例:**
- 「プロジェクト全体をチェックして」
- 「コミット前のファイルをチェックして」
- 「このファイルに個人情報が含まれていないか確認して」

**ステアリングファイル:** `.kiro/agents/privacy-guard/steering/guide.md`

## learning-tracker

学び記録エージェント

**役割:** 開発活動（コミット、プッシュ、コードレビュー）から得た学びや気づきを記録・蓄積し、成長を可視化する

**使用例:**
- 「今日の学びを記録して」
- 「先月の学びを見せて」
- 「今月の学びをまとめて」
- 「学びの統計を見せて」

**ステアリングファイル:** `.kiro/agents/learning-tracker/steering/guide.md`

## rails-code-reviewer

Ruby/Rails専門コードレビューエージェント

**役割:** Ruby言語とRails規約、パフォーマンス、セキュリティ、フロントエンド（Tailwind/Stimulus）、テスト品質（RSpec）を自動レビュー

**使用例:**
- 「このファイルをレビューして」
- 「app/controllers を全体的にレビューして」
- 「セキュリティ観点でチェックして」
- 「パフォーマンス問題を見つけて」
- 「テストの品質をチェックして」

**レビュー観点:**
- Ruby言語のベストプラクティス
- Rails規約（Fat Controller/Thin Model等）
- N+1クエリ、パフォーマンス最適化
- セキュリティ（Strong Parameters、XSS、SQLインジェクション）
- Tailwind/Stimulusのベストプラクティス
- RSpecテスト品質
- Rubocop準拠

**ステアリングファイル:** `.kiro/agents/rails-code-reviewer/steering/review-guide.md`
