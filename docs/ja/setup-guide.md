# MCP + OAuth 2.1 + AWS Cognito セットアップガイド

このガイドでは、AWS Cognitoを使用したMCP OAuth 2.1デモのセットアップと実行方法について説明します。このプロジェクトは、AWS Cognitoを認証サーバーとして使用したOAuth 2.1によるModel Context Protocol (MCP) 認証フローの完全な実装を実演します。

## 詳細な設定手順

### 前提条件

- IAM権限を持つAWSアカウント
- Node.js 18+ 
- 認証情報が設定されたAWS CLI
- OAuth 2.1とAWS Cognitoの基本的な理解

### 環境設定

1. **AWS認証情報**
   - AWS CLIを設定: `aws configure`
   - 以下を作成する権限があることを確認:
     * Cognito ユーザープール
     * IAM ロール
     * API Gateway リソース
     * Lambda 関数
     * DynamoDB テーブル

2. **リポジトリのクローン**

   ```bash
   git clone https://github.com/empires-security/mcp-oauth2-aws-cognito.git
   cd mcp-oauth2-aws-cognito
   ```

3. **Node.js依存関係**
   ```bash
   npm run install:all
   ```

4. **AWSリソースのデプロイ**

   ```bash
   npm run deploy
   ```

   問題なければ以下の様になるはず！

   ```bash
    Stack deployment successful! Here are the details:
    =================================================
    UserPoolClientId: <固有の値>
    ApiGatewayUrl: https://<固有の値>.execute-api.ap-northeast-1.amazonaws.com/v1
    UserPoolDomain: mcp-oauth-demo-domain-796032104877.auth.ap-northeast-1.amazoncognito.com
    UserPoolId: ap-northeast-1_BILaFaGkl
    DynamoDBTable: mcp-oauth-demo-dcr-clients
    RegisterClientEndpoint: https://<固有の値>.execute-api.ap-northeast-1.amazonaws.com/v1/register
    Created .env files for client, auto-client, and server
   ```

5. **環境設定**
   - 以下で生成された`.env`ファイルを確認:
     * `src/client/.env`
     * `src/auto-client/.env`
     * `src/mcp-server/.env`
   - `.env.example`ファイルと比較
   - 必要に応じてCLIENT_SECRETを手動で確認/更新

## アプリケーションの実行

### 1. サーバーとクライアントの同時実行

```bash
pnpm run dev
```

### 2. MCPサーバー

- MCPサーバーをローカルで実行
- Cognito認証サーバーへのリンクを含む`/.well-known/oauth-protected-resource`エンドポイントを提供することを確認

### 3. クライアント

- サンプルクライアントを実行
- 以下を行います:
  - 認証サーバーを発見
  - ブラウザを介してユーザーを認証にリダイレクト
  - アクセストークンを取得
  - 保護されたMCPサーバーエンドポイントにアクセス

## プロジェクト構造

```
/mcp-server
  - 最小限の保護リソースサーバー
  - /.well-known/oauth-protected-resourceを提供

/auto-client
  - Dynamic client registrationの例
  - 認証サーバーを自動発見し登録
  - 動的に取得した認証情報でOAuth 2.1フローを処理

/client
  - OAuth 2.1フローを実演するサンプルクライアント
  - 発見、PKCE、トークン使用を処理

/docs
  - セットアップガイドと説明

/infrastructure
  - AWSリソース用CloudFormationテンプレート
  - Dynamic Client Registration用API Gateway
  - クライアント登録管理用Lambda関数
  - クライアント登録保存用DynamoDB
```

### 4. OAuthフローのテスト

ブラウザで`http://localhost:3000`にアクセスし、以下の手順に従ってください:

1. "Log in"ボタンをクリック
2. 認証のためCognitoホストUIにリダイレクトされます
3. アカウントを作成するか、既存の認証情報でログイン
4. 認証が成功すると、クライアントアプリにリダイレクトされます
5. "Fetch MCP Data"をクリックして、MCPサーバーへの認証済みリクエストを作成

<br/>

環境変数にCognitoで生成されたクライアントシークレットを設定する必要があります。

`アプリケーションクライアント: mcp-oauth-demo-static-client` というものができているはずなのでマネジメントコンソールにアクセスしてそこから取得する必要があります。

### 5. DCR付き自動発見クライアント

ブラウザで`http://localhost:3002`にアクセスし、以下の手順に従ってください:

1. "Log in"ボタンをクリック
2. クライアントは以下を行います:
   - MCPサーバーと認証エンドポイントを自動発見
   - DCRエンドポイントに自己登録
   - Cognitoに認証でリダイレクト
3. 認証後、自動クライアントにリダイレクトされます
4. "Fetch MCP Data"をクリックして、動的登録されたクライアントで認証済みリクエストを作成

## アーキテクチャ概要
- 詳細な[アーキテクチャ概要](./architecture-guide.md)

### コンポーネント

1. **MCPクライアント**: 以下を行うNode.js Expressアプリケーション:
   - Protected Resource Metadataを使用して認証サーバーを発見
   - PKCEを使用したOAuth 2.1認証コードフローを実装
   - MCPサーバーに認証済みリクエストを作成

2. **MCPサーバー**: 以下のAWS API Gateway + Lambdaとして実装:
   - Protected Resource Metadataドキュメントを提供
   - Cognitoからのアクセストークンを検証
   - MCP操作用の保護されたAPIエンドポイントを提供

3. **AWS Cognito**: OAuth 2.1認証サーバーとして機能:
   - ユーザー認証を処理
   - アクセストークンとリフレッシュトークンを発行
   - 認証サーバーメタデータを提供

### 認証フロー

フローはMCP認証仕様で説明されているシーケンスに従います:

1. クライアントがトークンなしでMCPサーバーにリクエスト
2. サーバーが401 + リソースメタデータを指すWWW-Authenticateヘッダーでレスポンス
3. クライアントがリソースメタデータを取得して認証サーバーを発見
4. クライアントが認証のためユーザーを認証サーバー（Cognito）にリダイレクト
5. ユーザーがCognitoで認証し、認証コードでリダイレクトされて戻る
6. クライアントがPKCEフローを使用してコードをトークンと交換
7. クライアントがアクセストークンを使用してMCPサーバーに認証済みリクエストを作成
8. サーバーがトークンを検証し、保護されたリソースへのアクセスを許可

## プロジェクト構造

```
mcp-oauth2-aws-cognito/
├── infrastructure/            # AWSインフラストラクチャコード
│   └── cloudformation/        # CloudFormationテンプレート
├── scripts/                   # デプロイスクリプト
├── src/
│   ├── mcp-server/            # MCPサーバー実装
│   ├── auto-client/           # MCP自動発見クライアント実装
│   ├── client/                # MCPクライアント実装
│   └── shared/                # 共有ユーティリティ
└── docs/                      # ドキュメント
```

## クリーンアップ

このデモ用に作成されたすべてのAWSリソースを削除するには:

```bash
npm run cleanup
```

これにより以下が削除されます:
- Cognitoユーザープールとアプリクライアント
- 関連するIAMロールとポリシー

## トラブルシューティング

### よくある問題

1. **401 Unauthorizedだが、WWW-Authenticateヘッダーがない**
   - MCPサーバー実装がWWW-Authenticateヘッダーを正しく設定していることを確認
   - 形式がRFC9728要件に従っていることを確認

2. **トークン検証の失敗**
   - Cognitoクライアントアプリが正しいコールバックURLで設定されていることを確認
   - PKCEがクライアント側で適切に実装されていることを確認
   - 要求されたスコープがCognitoで設定されたものと一致することを確認

3. **CORS問題**
   - CORSエラーが発生している場合、API Gateway CORS設定を確認
   - OPTIONSリクエストが適切に処理されていることを確認

4. **CloudFormationデプロイの失敗**
   - AWS CLIに十分な権限があることを確認
   - 詳細なエラーメッセージについてCloudFormationイベントを確認

5. **Dynamic Client Registration問題**
   - 登録エラーについてAPI Gatewayログを確認
   - Lambda関数がCognitoクライアントを作成する適切な権限を持っていることを確認
   - DynamoDBテーブルが存在し、アクセス可能であることを確認
   - 自動クライアントを使用している場合、詳細な発見と登録エラーについてコンソールログを確認

## 高度な設定

### カスタムドメイン

デフォルトのCognitoとAPI Gateway URLの代わりにカスタムドメインを使用するには:

1. AWS Route 53または好みのドメインレジストラでドメインを登録
2. ドメイン用のACM証明書を作成
3. Cognitoユーザープール用のカスタムドメインを設定
4. API Gateway用のカスタムドメインを設定

### Cognitoユーザープール設定

追加のセキュリティまたはカスタマイズのため:

1. より強力な認証のためMFAを有効化
2. ブランディングでCognitoホストUIをカスタマイズ
3. 追加のアイデンティティプロバイダー（Google、Facebookなど）を追加
4. カスタムメールまたはSMSメッセージを設定

### 複数の認証サーバーでのテスト

MCP仕様は複数の認証サーバーをサポートします。これをテストするには:

1. 別のOAuthプロバイダー（Auth0、Oktaなど）を設定
2. Protected Resource Metadataの`authorization_servers`配列に追加
3. 複数の認証サーバー間の選択を処理するようクライアントを更新

### Dynamic Client Registrationセキュリティ

本番環境では、以下でDCRセキュリティを強化:

1. **初期アクセストークン**
   - クライアント登録に事前認証トークンを要求
   - API Gatewayオーソライザーを使用してトークンを検証

2. **クライアント認証**
   - API Gatewayエンドポイント用のmutual TLS (mTLS)を実装
   - 登録リクエストにクライアント証明書を要求

3. **登録ポリシー**
   - 新しいクライアントの承認ワークフローを実装
   - 許可されたリダイレクトURIを信頼できるドメインに制限
   - 動的登録されたクライアントが利用可能なスコープを制限

4. **レート制限**
   - API Gateway使用プランとAPIキーを追加
   - 異常な登録パターンのCloudWatchアラームを実装
