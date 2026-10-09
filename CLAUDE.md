# CLAUDE.md

このリポジトリの運用ルール（モード・テスト形式・コミュニケーション方針）は AGENTS.md を正とし、ここで読み込みます。

@AGENTS.md

以下は Claude Code 固有の補足だけです。ルールを変えるときは AGENTS.md / SKILL.md 側を直し、ここに同じ内容を書き写さないでください。

## インプットモード: スキルの読み込み
- Claude Code は `.agents/skills/` を自動検出しません。AGENTS.md の「`knowledge-input` スキルを読み込む」は、**`.agents/skills/knowledge-input/SKILL.md` を Read ツールで読み、その登録フローに従う**という意味です。
- 単語が投稿されたら、毎回そのファイルを読んでから始めてください（記憶だけで進めない）。

## data.json の編集
実ファイルの書式（これに合わせる）:
- トップレベルは配列。各要素のキーは次の6つで、順序もこのまま: `term`, `summary`, `how_to_use`, `status`, `created_at`, `last_tested_at`（README のデータ構造は4項目だけの古い記述）
- インデントはスペース2つ。日本語は `\uXXXX` にエスケープせずそのまま書く。ファイル末尾は改行1つ。
- 日時は JST の ISO 8601 で秒まで書く: `"2026-10-06T00:25:20+09:00"`。未テストの `last_tested_at` は `null`。
- `status` は `new` / `reviewing` / `learned` のどれか。新規登録は `status: "new"`・`last_tested_at: null` で配列の末尾に追加する。
- テスト後は `status` と `last_tested_at` だけを更新する（`created_at` などは変えない）。

注意点:
- 実行環境の時計は UTC のことがあります。時刻は必ず JST で取ってください: `TZ=Asia/Tokyo date +%Y-%m-%dT%H:%M:%S%:z`
- 編集は Edit ツールで該当箇所だけ書き換えるのが基本です。Python で丸ごと書き戻す場合は `json.dumps(d, ensure_ascii=False, indent=2) + "\n"` で書いてください（この設定なら既存ファイルとバイト単位で一致します）。
- 重複調査やテスト候補選びでは、全文を読む前に一覧を出すと早いです:
  `python3 -c "import json; [print(e['status'], e['term']) for e in json.load(open('data.json'))]"`
- 編集後は、コミット前に必ず次の検証を通してください:

```sh
python3 - <<'EOF'
import json, re
s = open('data.json', encoding='utf-8').read()
d = json.loads(s)
K = ['term', 'summary', 'how_to_use', 'status', 'created_at', 'last_tested_at']
T = re.compile(r'\d{4}-\d\d-\d\dT\d\d:\d\d:\d\d\+09:00$')
assert all(list(e) == K for e in d), 'キーの過不足・順序違い'
assert all(e['status'] in ('new', 'reviewing', 'learned') for e in d), '不正な status'
assert all(T.match(e['created_at']) and (e['last_tested_at'] is None or T.match(e['last_tested_at'])) for e in d), '日時の書式違い'
assert s == json.dumps(d, ensure_ascii=False, indent=2) + '\n', 'インデント・エスケープ・末尾改行のずれ'
print(len(d), 'entries OK')
EOF
```

## コミット
- AGENTS.md と SKILL.md が「用語の登録・テスト結果の更新ごとにコミットする」と指示しているので、これをユーザーからのコミット指示として扱ってください。ユーザーが承認した登録や、テストの判定が済んだら、改めて確認を取らずにコミットしてかまいません。
- コミットには、その操作で変えたファイル（通常は `data.json` だけ）を含めます。
- SKILL.md が参照している `git-commit-convention` スキルは、このリポジトリには含まれていません。使える環境ではそれに従い、見つからない場合は既存の git log の慣習に合わせてください:
  - 形式: `<type>(<scope>): <日本語の要約>`、空行、本文（なぜ・何をしたか）と `- ` の箇条書き
  - 用語の登録: `feat(knowledge): …を追加`。箇条書きで登録した用語を並べ、重複でスキップした用語があればそれも書く
  - 既存の用語の定義を直したとき: `docs(knowledge): …`
  - テスト結果: `docs(study): <用語> のテスト結果を記録（<新しい status>）`。本文に `対象単語` / `テスト形式`（A か C）/ `結果` を箇条書きで書く
  - スキルや運用ルールの変更: `chore(skills): …`、設定ファイル: `chore(settings): …`
