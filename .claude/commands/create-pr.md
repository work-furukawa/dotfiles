# Claude コマンド: Create PR

このコマンドは、現在のブランチの変更内容を元にGitHub PRをdraftで作成します。

## 使用方法

```
/create-pr [base-branch]
```

base-branchが省略された場合は、リポジトリのデフォルトブランチを自動検出します。

## このコマンドの機能

1. **現在のブランチを確認** - PRを作成するブランチを特定
2. **デフォルトブランチ検出** - base-branchが指定されていない場合、リモートのデフォルトブランチを自動検出
3. **リモート状態確認** - ブランチがリモートにプッシュされているかチェック（未プッシュの場合は確認メッセージを表示）
4. **差分を解析** - base-branchとの差分からコミット内容を分析
5. **タイトル生成** - 変更内容を要約した簡潔なタイトルを自動生成
6. **PRテンプレート適用** - リポジトリの.github/pull_request_template.mdまたは.github/PULL_REQUEST_TEMPLATE.mdを使用
7. **Draft PR作成** - GitHub CLI (gh)を使用してdraft PRを作成

## 前提条件

- 現在のディレクトリがGitリポジトリであること
- GitHubリポジトリに接続されていること
- GitHub CLI (gh)がインストール・認証済みであること
- feature branchにいること（main/masterブランチではないこと）

## 自動生成されるPRタイトルの規則

コミットメッセージとdiffを分析し、以下の形式でタイトルを生成：
- **feat:** 新機能追加の場合
- **fix:** バグ修正の場合
- **refactor:** リファクタリングの場合
- **docs:** ドキュメント更新の場合
- **test:** テスト追加・修正の場合
- **style:** スタイル変更の場合
- **chore:** その他の変更

## PRテンプレートの検索順序

以下の順序でPRテンプレートを検索し、最初に見つかったものを使用：
1. `.github/pull_request_template.md`
2. `.github/PULL_REQUEST_TEMPLATE.md`
3. `.github/pull_request_template/`ディレクトリ内の.mdファイル
4. テンプレートが見つからない場合はデフォルトテンプレートを使用

## 使用例

```bash
# デフォルトブランチ（main/master）をベースにPR作成
/create-pr

# developブランチをベースにPR作成
/create-pr develop

# 特定のブランチをベースにPR作成
/create-pr release/v1.0
```

## 実行される処理の流れ

1. **ブランチ確認**: 現在のブランチがデフォルトブランチ以外であることを確認
2. **デフォルトブランチ検出**: base-branchが未指定の場合、以下のコマンドでデフォルトブランチを検出:
   - `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`
   - または `git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'`
3. **リモート状態確認**: ブランチがリモートに存在するかチェック
   - 未プッシュの場合: 「ブランチをプッシュしてください: `git push -u origin <branch>`」メッセージを表示して停止
   - プッシュ済みの場合: 処理を続行
4. **差分解析**: `git diff <base-branch>...HEAD` でファイル変更を分析
5. **コミット解析**: `git log <base-branch>..HEAD --oneline` でコミット履歴を分析
6. **タイトル生成**: 変更内容に基づいて適切なプレフィックス付きタイトルを生成
7. **テンプレート取得**: PRテンプレートファイルを検索・読み込み
8. **PR作成**: `gh pr create --draft --title "<title>" --body "<template>"` を実行

## エラーハンドリング

- デフォルトブランチにいる場合は警告を出して処理を停止
- GitHubリポジトリでない場合は適切なエラーメッセージを表示
- ブランチが未プッシュの場合は適切なpushコマンドを提示して停止
- PRテンプレートが見つからない場合はデフォルトテンプレートを使用

## 注意事項

- このコマンドは**draft PR**を作成します
- **自動pushは行いません** - 事前に手動でブランチをプッシュしてください
- 既存のPRがある場合は確認メッセージを表示
- 生成されたタイトルは後から手動で編集可能
- PRテンプレートの内容はそのまま使用されるため、必要に応じて後から編集してください

## 事前準備

PR作成前に以下を実行してください：
```bash
# 変更をコミット
git add .
git commit -m "your commit message"

# ブランチをリモートにプッシュ
git push -u origin <your-branch-name>

# その後、PR作成
/create-pr
```
