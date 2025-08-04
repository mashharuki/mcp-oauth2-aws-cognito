# タスク完了時の推奨アクション

## コード変更後のチェックリスト

### 1. コード品質確認
- [ ] 変更したファイルに構文エラーがないか確認
- [ ] コンソールエラーが発生していないか確認
- [ ] 関連する環境変数が正しく設定されているか確認

### 2. 機能テスト
- [ ] 変更した機能が正常に動作するか確認
- [ ] OAuth認証フローが正常に完了するか確認
- [ ] MCP APIエンドポイントが正常にレスポンスを返すか確認

### 3. 統合テスト
```bash
# アプリケーション全体の動作確認
npm run dev

# 各コンポーネントの個別テスト
npm run start:server      # サーバーが起動するか
npm run start:client      # クライアントが起動するか  
npm run start:auto-client # 自動発見クライアントが起動するか
```

### 4. セキュリティチェック
- [ ] 機密情報がコードにハードコードされていないか確認
- [ ] 環境変数ファイル (`.env`) が `.gitignore` に含まれているか確認
- [ ] JWT トークン検証が適切に実装されているか確認

### 5. ドキュメント更新
- [ ] 新機能や変更点を `README.md` に反映
- [ ] 設定変更がある場合は `docs/setup-guide.md` を更新
- [ ] 環境変数の追加・変更がある場合は `.env.example` を更新

### 6. 本番デプロイ前確認
```bash
# AWSリソースの状態確認
aws cloudformation describe-stacks --stack-name mcp-oauth-demo

# 環境設定の再確認
cat src/mcp-server/.env
cat src/client/.env
cat src/auto-client/.env
```

### 7. クリーンアップ (開発完了時)
```bash
# AWSリソースのクリーンアップ
npm run cleanup

# 一時ファイルの削除
rm -rf node_modules
rm -rf src/*/node_modules
```

## 緊急時対応

### サーバーが起動しない場合
1. 依存関係の再インストール: `npm run install:all`
2. 環境変数の確認: `.env` ファイルの内容チェック
3. ポート競合の確認: `lsof -i :3001` (サーバー用)

### 認証エラーが発生する場合
1. AWS Cognito設定の確認
2. JWT署名の検証ロジック確認
3. 環境変数 `COGNITO_USER_POOL_ID`, `COGNITO_CLIENT_ID` の確認

### CloudFormationエラーの場合
1. AWS認証情報の確認: `aws sts get-caller-identity`
2. 権限の確認: IAM ポリシーのチェック
3. スタック状態の確認: AWS コンソールでスタック詳細確認