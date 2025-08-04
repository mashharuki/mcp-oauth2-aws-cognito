# 技術スタック詳細

## メイン技術
- **Runtime**: Node.js 18+
- **Backend Framework**: Express.js 5.x
- **言語**: JavaScript (ES6+)
- **パッケージマネージャー**: npm

## 主要依存関係

### MCP Server
- **express**: 5.1.0 - Webアプリケーションフレームワーク
- **axios**: 1.9.0 - HTTP クライアント
- **jsonwebtoken**: 9.0.2 - JWT処理
- **jwk-to-pem**: 2.0.7 - JWK からPEM形式への変換
- **cors**: 2.8.5 - CORS対応
- **dotenv**: 16.5.0 - 環境変数管理

### クライアント (共通)
- **express**: 5.1.0 - ローカルサーバー
- **axios**: 1.9.0 - HTTP クライアント
- **express-session**: 1.18.1 - セッション管理
- **crypto**: 1.0.1 - 暗号化処理 (PKCE)
- **dotenv**: 16.5.0 - 環境変数管理
- **open**: 10.1.1 - ブラウザ自動起動（clientのみ）

### AWS インフラ
- **@aws-sdk/client-cloudformation**: 3.798.0 - CloudFormation SDK
- **AWS Cognito**: ユーザープール認証
- **CloudFormation**: インフラストラクチャ管理

### 開発ツール
- **concurrently**: 9.1.2 - 複数プロセス同時実行

## ディレクトリ構造
```
src/
├── mcp-server/     # リソースサーバー実装
├── client/         # 静的クライアント実装  
├── auto-client/    # 自動発見クライアント実装
└── shared/         # 共通設定ファイル
docs/               # アーキテクチャ・セットアップガイド
infrastructure/     # CloudFormationテンプレート
scripts/            # デプロイ・クリーンアップスクリプト
```