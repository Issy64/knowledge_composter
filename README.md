# Knowledge Composter

日々の業務や学習で見つけた「気になる単語」「よくわからない用語」を放り込み、AIとの対話テストを通じて知識として定着させるための「ナレッジバケツ」リポジトリです。

## データ構造 (data.json)
- `term`: 用語
- `summary`: 意味・WHAT
- `how_to_use`: 使い方・HOW
- `status`: 学習状況（new, reviewing, learned）
- `created_at`: 登録日時（JST の ISO 8601 形式。例: 2026-10-06T00:25:20+09:00）
- `last_tested_at`: 最終テスト日時（形式は `created_at` と同じ。未テストなら null）
