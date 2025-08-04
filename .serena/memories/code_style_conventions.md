# コードスタイル・規約

## 全般的なスタイル
- **言語**: JavaScript (ES6+)
- **モジュールシステム**: CommonJS (`require`/`module.exports`)
- **非同期処理**: async/await パターンを優先使用
- **エラーハンドリング**: try-catch ブロックで適切に処理

## 命名規約
- **変数・関数**: camelCase (例: `userPoolId`, `validateToken`)
- **定数**: UPPER_SNAKE_CASE (例: `PORT`, `STACK_NAME`)
- **ファイル名**: kebab-case (例: `auth-flow.js`, `resource-metadata.js`)
- **環境変数**: UPPER_SNAKE_CASE (例: `COGNITO_USER_POOL_ID`)

## ファイル構造パターン
### MCPサーバー (`src/mcp-server/`)
- `index.js`: メインのExpressアプリケーション
- `resource-metadata.js`: PRM(Protected Resource Metadata)生成
- `token-validator.js`: JWT トークン検証ロジック

### クライアント共通パターン
- `index.js`: メインアプリケーション
- `auth-flow.js`: OAuth認証フロー処理
- `discovery.js`: 動的発見ロジック  
- `mcp-api.js`: MCP API呼び出し

## Express.js パターン
```javascript
// ミドルウェア定義
app.use(cors());
app.use(express.json());

// エンドポイント定義
app.get('/endpoint', async (req, res) => {
  try {
    // 処理ロジック
    res.json(result);
  } catch (error) {
    console.error('Error:', error.message);
    res.status(500).json({ error: 'server_error' });
  }
});
```

## エラーレスポンス形式
OAuth 2.1 仕様に準拠:
```javascript
res.status(401).json({
  error: 'unauthorized',
  error_description: 'Valid bearer token required'
});
```

## 環境変数パターン
```javascript
// デフォルト値を提供
const PORT = process.env.PORT || 3001;
const REGION = process.env.AWS_REGION || 'us-east-1';
```

## ログ出力
- `console.log()` でデバッグ情報
- `console.error()` でエラー情報
- 重要な処理の開始・完了をログ出力

## セキュリティ考慮事項
- 秘密鍵や機密情報は環境変数で管理
- JWTトークンの署名検証を必須実装
- CORS設定を適切に構成
- PKCE (Proof Key for Code Exchange) を OAuth フローで使用