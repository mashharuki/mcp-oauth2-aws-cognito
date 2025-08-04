# 推奨コマンド

## インストールコマンド
```bash
# すべての依存関係をインストール
npm run install:all

# 個別インストール
npm run install:client      # clientの依存関係のみ
npm run install:auto-client # auto-clientの依存関係のみ 
npm run install:server      # mcp-serverの依存関係のみ
```

## デプロイ・インフラコマンド
```bash
# AWS Cognitoリソースをデプロイ
npm run deploy

# リソースをクリーンアップ
npm run cleanup
```

## 実行コマンド
```bash
# 各コンポーネントを個別に起動
npm run start:server      # MCPサーバー起動 (ポート3001)
npm run start:client      # クライアント起動 (ポート3000)
npm run start:auto-client # 自動発見クライアント起動 (ポート3002)

# 同時実行 (開発時推奨)
npm run dev              # 全コンポーネントを同時起動
```

## 開発ワークフロー
1. `npm run install:all` - 依存関係をインストール
2. `npm run deploy` - AWS リソースをデプロイ
3. `.env` ファイルを確認・調整
4. `npm run dev` - アプリケーション起動
5. 開発完了後: `npm run cleanup` - リソースクリーンアップ

## システムコマンド (macOS)
```bash
# プロジェクト探索
find . -name "*.js" -type f        # JSファイル検索
grep -r "express" src/              # ソース内テキスト検索
ls -la                              # ファイル一覧
cd src/mcp-server && ls             # ディレクトリ移動・一覧

# Git操作
git status                          # ステータス確認
git log --oneline                   # コミット履歴
git diff                           # 変更差分
```

## 環境設定ファイル
- `src/client/.env` - クライアント設定
- `src/auto-client/.env` - 自動発見クライアント設定  
- `src/mcp-server/.env` - MCPサーバー設定
- 各ディレクトリに `.env.example` が参考として配置