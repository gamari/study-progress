# GitとGitHub CLIの基本

## Gitの基本コマンド

Gitはバージョン管理システムで、コードの変更を追跡します。

- `git init`: 現在のディレクトリをGitリポジトリとして初期化します。
- `git add .`: ワークスペースのすべての変更をステージングエリアに追加します。
- `git commit -m "メッセージ"`: ステージングされた変更をコミットします。メッセージは変更の説明です。
- `git status`: リポジトリの現在の状態（変更されたファイル、ステージングされたファイルなど）を表示します。
- `git log`: コミットの履歴を表示します。
- `git branch`: ブランチの一覧を表示します。`-m`オプションでブランチ名を変更できます。
- `git checkout <ブランチ名>`: 指定したブランチに切り替えます。
- `git merge <ブランチ名>`: 指定したブランチを現在のブランチにマージします。
- `git push`: ローカルのコミットをリモートリポジトリにプッシュします。
- `git pull`: リモートリポジトリの変更をローカルにプルします。

## GitHub CLIの基本

GitHub CLI (gh) は、GitHubをコマンドラインから操作するためのツールです。

- `gh auth login`: GitHubアカウントで認証します。初回使用時に必要です。
- `gh repo create <リポジトリ名>`: 新しいGitHubリポジトリを作成します。`--public`で公開、`--private`で非公開。
- `gh repo clone <リポジトリ>`: GitHubリポジトリをローカルにクローンします。
- `gh issue list`: リポジトリのイシュー一覧を表示します。
- `gh pr create`: プルリクエストを作成します。
- `gh repo view`: リポジトリの情報を表示します。

## 今回の学習内容

- GitHub CLIのインストール: `sudo apt install gh`
- 認証: `gh auth login` でGitHubアカウントにログイン。
- リポジトリ作成と接続: `gh repo create study-progress --public --source=. --remote=origin --push` でローカルディレクトリからGitHubリポジトリを作成し、接続してプッシュ。
- Gitの初期化: `git init` でリポジトリ初期化、`git add . && git commit -m "Initial commit"` で初期コミット。

これらのコマンドを使って、プロジェクトのバージョン管理とGitHubとの連携が可能です。