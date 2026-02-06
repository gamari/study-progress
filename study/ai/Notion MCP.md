# Overview

[Home](https://www.notion.so/notion/Notion-MCP-1d0efdeead058054a339ffe6b38649e1)


## AIによる手順

1. Notion 側でワークスペース／ページ／データベースを整理しておく
2. Notion MCP サーバー（または自ホスト版）を使って、特定のページ・スペースを “MCP の読み込み対象” に設定します。 ([Notion][1])
3. AI ツール（Cursor, Claude など）側で「Notion MCP 経由でデータアクセス」設定を行う（例：OAuth 認証、ページ選択） ([Notion][1])
4. AI ツールは指定された Notion 内の内容を読み込み、プロンプトの一部として活用したり、リアルタイムにコンテキスト参照を行ったりできます。
5. 要件・設計・ドキュメントが更新されれば、AI ツールも最新の内容を参照できるよう設計することで、より正確・最新の支援が可能になります。

## VSCodeの設定手順

- Notionの設定
  - Integrationを作成
  - Integrationに対して接続できるDBを追加していく
- VSCodeの設定
  - ctrl + shift + p
  - MCP: Add Server...を選択
  - NPM Packageを選択
  - @notionhq/notion-mcp-server
  - mcp.json内で「OPENAPI_MCP_HEADERS」をIntegrationのものに指定
  - Agent内のリロードボタンを押下
  - Add Contextで「Notion API」を指定してあげる
  - 接続できることを確認


## メモ

- Notion MCPで色々な要素に接続させる
- タスクをdocsで書く
  - それを支持させる