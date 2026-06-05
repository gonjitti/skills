# イシュートラッカー: GitHub

このリポジトリのイシューおよびPRDは、GitHub Issuesで管理されます。すべての操作で `gh` CLIを使用します。

## 操作規則（コマンド例）

- **イシューの作成**: `gh issue create --title "..." --body "..."`。複数行の本文にはヒアドキュメントを使用します。
- **イシューの読み取り**: `gh issue view <番号> --comments`。`jq` でコメントをフィルタリングし、ラベルも取得します。
- **イシューのリスト表示**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`。必要に応じて `--label` や `--state` フィルターを適用します。
- **イシューへのコメント**: `gh issue comment <番号> --body "..."`
- **ラベルの付与 / 削除**: `gh issue edit <番号> --add-label "..."` / `--remove-label "..."`
- **イシューのクローズ**: `gh issue close <番号> --comment "..."`

`git remote -v` からリポジトリ情報を推測します。`gh` はクローンされたリポジトリ内で実行されると自動的にこれを判別します。

## スキルが「イシュートラッカーに公開する」と指示した場合

GitHubのイシューを作成してください。

## スキルが「関連チケットを取得する」と指示した場合

`gh issue view <番号> --comments` を実行してください。
