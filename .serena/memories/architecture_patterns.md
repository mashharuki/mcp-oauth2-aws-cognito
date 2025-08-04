# アーキテクチャ・設計パターン

## 全体アーキテクチャ
このプロジェクトは **プロバイダーに依存しない OAuth 2.1 実装** を特徴とする3層アーキテクチャです。

### 1. Resource Server (MCPサーバー)
- **場所**: `src/mcp-server/`
- **役割**: 保護されたリソース (MCP API) を提供
- **主要パターン**:
  - Express.js ミドルウェアパターン
  - JWT トークン検証
  - 動的メタデータプロキシ

### 2. OAuth Client Layer
- **静的クライアント** (`src/client/`): 事前設定されたクレデンシャル使用
- **自動発見クライアント** (`src/auto-client/`): Dynamic Client Registration (DCR) 対応

### 3. Authorization Server
- **例**: AWS Cognito (任意のOAuth 2.1準拠サーバーに置換可能)

## 設計パターン

### Discovery Pattern (発見パターン)
```javascript
// 1. 保護リソースメタデータ発見
GET /.well-known/oauth-protected-resource

// 2. 認証サーバーメタデータ発見  
GET /.well-known/oauth-authorization-server

// 3. 動的エンドポイント取得
// 全てランタイムで発見、ハードコード無し
```

### Proxy Pattern (プロキシパターン)
```javascript
// MCPサーバーが認証サーバーメタデータをプロキシ
app.get('/.well-known/oauth-authorization-server', async (req, res) => {
  const response = await axios.get(cognitoMetadataUrl);
  const metadata = response.data;
  // DCR エンドポイント追加など拡張
  metadata.registration_endpoint = process.env.DCR_ENDPOINT;
  res.json(metadata);
});
```

### Token Validation Pattern
```javascript
// 動的 JWKS 発見と検証
const validateToken = async (token) => {
  // 1. JWT ヘッダーから kid を取得
  // 2. JWKS URI から公開鍵を動的取得
  // 3. 署名検証 + クレーム検証
};
```

### PKCE Security Pattern
```javascript
// OAuth 2.1 PKCE フロー
// 1. code_verifier 生成 (暗号学的にセキュア)
// 2. code_challenge = base64url(sha256(code_verifier))
// 3. 認証要求時に code_challenge 送信
// 4. トークン交換時に code_verifier 送信
```

## 重要な設計原則

### 1. Provider Agnostic (プロバイダー非依存)
- ハードコードされた認証サーバー固有のエンドポイント無し
- 全てメタデータ発見による動的設定
- OAuth 2.1 標準準拠によるポータビリティ

### 2. Security First
- PKCE による認証コード攻撃対策
- JWTの署名検証必須
- 適切な WWW-Authenticate ヘッダー応答

### 3. MCP 2025-06-18 仕様準拠
- Protected Resource Metadata (PRM) サポート
- Resource Indicators (RFC 8707) サポート
- Dynamic Client Registration (DCR) サポート

### 4. Express.js ベストプラクティス
- ミドルウェアチェーン活用
- エラーハンドリング統一
- CORS 適切設定
- セッション管理