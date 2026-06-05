# イシュートラッカー: GitLab

このリポジトリのイシューおよびPRDは、GitLab Issuesで管理されます。すべての操作で [`glab`](https://gitlab.com/gitlab-org/cli) CLIを使用します。

## 操作規則（コマンド例）

- **イシューの作成**: `glab issue create --title "..." --description "..."`。複数行の説明文にはヒアドキュメントを使用します。エディタを開く場合は `--description -` を渡します。
- **イシューの読み取り**: `glab issue view <番号> --comments`。機械可読な出力を得るには `-F json` を使用します。
- **イシューのリスト表示**: 適切な `--label` フィルターを指定して `glab issue list -F json` を実行します。
- **イシューへのコメント**: `glab issue note <番号> --message "..."`。GitLabではコメントを「ノート（note）」と呼びます。
- **ラベルの付与 / 削除**: `glab issue update <番号> --label "..."` / `--unlabel "..."`。複数のラベルはカンマ区切りにするか、フラグを繰り返すことで指定できます。
- **イシューのクローズ**: `glab issue close <番号>`。`glab issue close` はクローズ時のコメントを受け付けないため、先に `glab issue note <番号> --message "..."` で説明文を投稿してからクローズしてください。
- **マージリクエスト**: GitLabではプルリクエスト（PR）を「マージリクエスト（MR）」と呼びます。`glab mr create`、`glab mr view`、`glab mr note` などを使用します。`gh pr ...` コマンドの `pr` を `mr` に、`comment`/`--body` を `note`/`--message` に置き換えたものと同等です。

`git remote -v` からリポジトリ情報を推測します。`glab` はクローンされたリポジトリ内で実行されると自動的にこれを判別します。

## スキルが「イシュートラッカーに公開する」と指示した場合

GitLabのイシューを作成してください。

## スキルが「関連チケットを取得する」と指示した場合

`glab issue view <番号> --comments` を実行してください。
