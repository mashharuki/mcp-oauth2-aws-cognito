# MCP + OAuth2.1 + AWS Cognito サンプル

## 概要

このリポジトリは、Node.jsとExpress.jsで完全に実装された、OAuth 2.1認証フローを使用してModel Context Protocol (MCP) サーバーを保護する方法を実演します。この例ではAWS Cognitoを認証サーバーとして使用していますが、実装は**プロバイダーに依存しない**設計で、任意のOAuth 2.1準拠の認証サーバーと連携可能です。

MCP Authorization Specification (バージョン2025-06-18)に基づき、このプロジェクトでは以下を紹介します：
- 一般的なOAuthエンドポイントを持つ**リソースサーバー** (RS)として機能するMCPサーバー
- プロバイダーに依存しないOAuth 2.1実装（例としてAWS Cognitoを使用）
- PKCEとRFC 8707 リソースインジケーターを使用したOAuth 2.1認証コードフロー
- Protected Resource Metadata (PRM) ドキュメント発見
- **完全に動的な**認証サーバーメタデータ発見
- Dynamic Client Registration (DCR) サポート
- MCP 2025-06-18仕様による強化されたセキュリティ機能
- 2つのクライアント実装：
  - 事前設定済み認証情報を使用する静的クライアント
  - 動的登録機能付き自動発見クライアント

## プロバイダー非依存設計

この実装は、OAuth 2.1標準に従って任意の準拠認証サーバーとの互換性を確保しています：
- **MCPサーバー**: 標準的なOAuthメタデータエンドポイントを公開し、バッキング認証サーバーにプロキシ
- **クライアント**: ハードコードされたプロバイダー固有のロジックなしに、認証サーバーを動的に発見
- **トークン検証**: 発見されたJWKS URIと認証サーバーメタデータからの発行者情報を使用
- **柔軟なバックエンド**: Cognitoを例として使用していますが、任意のOAuth 2.1サーバーで代替可能

## 新しいMCP認証仕様の理解

新しいMCP認証仕様では、リソースサーバーと認証サーバーの明確な分離を導入し、AWS Cognito、Okta、Auth0などの既存アイデンティティプロバイダーとの統合を容易にしています。

仕様の主要コンポーネント：

1. **Protected Resource Metadata (PRM)** ドキュメント
   - MCPサーバーが`/.well-known/oauth-protected-resource`でこのドキュメントを提供
   - 認証サーバー、サポートされるスコープなどの情報を含む
   - RFC9728 (OAuth 2.0 Protected Resource Metadata)に準拠

2. **発見プロセス**
   - クライアントが401 Unauthorizedレスポンスを受信すると、WWW-AuthenticateヘッダーにPRMドキュメントへのポインターが含まれる
   - クライアントがPRMドキュメントを取得して認証サーバーURLを発見
   - クライアントが発見されたURLから認証サーバーメタデータを動的に取得（ハードコードされたエンドポイントなし）

3. **OAuth 2.1認証**
   - PKCEを使用した認証コードフロー
   - 認証されたリクエストでのベアラートークン使用
   - 発見されたJWKS URIと発行者情報を使用した動的トークン検証

4. **Dynamic Client Registration (DCR)**
   - クライアントが新しいMCPサーバーに自動的に登録することを許可
   - 手動クライアント登録プロセスの必要性を排除
   - 新しいサービスへのシームレスな発見と接続を実現
   - MCP仕様では「クライアントは事前にMCPサーバーセットを知らない」ため、DCRの実装を強く推奨
   - OAuth クライアント認証情報を取得する標準化された方法を提供
   - RFC7591 (OAuth 2.0 Dynamic Client Registration Protocol)に準拠

この実装では、これらの概念をプロバイダー非依存の方法で適用する方法を紹介しています。この例では、API GatewayエンドポイントとLambda関数を通じたカスタムDynamic Client Registrationを備えたAWS Cognitoを使用していますが、コアOAuthフローは任意の準拠認証サーバーで動作します。

## アーキテクチャ
```
クライアント → MCPサーバー → 認証サーバー (例: AWS Cognito)
            (リソースサーバー)    (OAuth 2.1プロバイダー)
```
1. クライアントがトークンなしでリクエストを送信
2. MCPサーバーが401 Unauthorized + PRMメタデータを指すWWW-Authenticateヘッダーでレスポンス
3. クライアントがPRMを取得し、認証サーバーURLを動的に発見
4. クライアントが認証サーバーメタデータを取得し、OAuth 2.1認証コードフロー（PKCE付き）を実行
5. クライアントがアクセストークンを取得し、MCPサーバーへのリクエストを再試行
6. MCPサーバーが動的に発見されたJWKSを使用してトークンを検証し、保護されたリソースへのアクセスを許可

詳細な概要については、[アーキテクチャ概要](./architecture-guide.md)を参照してください。

図表：
- [アーキテクチャ図](../mcp-oauth-architecture.mermaid)
- [シーケンス図](../mcp-oauth-sequence.mermaid)
- [DCRシーケンス図](../mcp-oauth-sequence-dcr.mermaid)

## Dynamic Client Registration (DCR)

この実装には、OAuth 2.1 Dynamic Client Registrationのサポートが含まれており、クライアントが以下を行えます：

1. MCPサーバーと認証エンドポイントを動的に発見
2. 認証サーバーに自己登録
3. OAuthフローのための認証情報を取得

DCRフローは以下のように動作します：

1. クライアントがMCPサーバーの保護リソースメタデータを発見
2. クライアントが認証サーバー（Cognito）を発見
3. クライアントがAPI GatewayのDCRエンドポイントに登録
4. 登録でCognitoアプリクライアントが作成され、認証情報が返される
5. クライアントがこれらの認証情報を標準OAuth 2.1フローで使用

**実装ノート**: AWS CognitoはOAuth 2.0 DCR (RFC7591)で指定されたDynamic Client Registrationをネイティブサポートしていません。この実装では以下を使用してこのギャップを埋めています：
- DCR APIインターフェースを提供するAPI Gatewayエンドポイント
- Cognitoアプリクライアントをプログラム的に作成するLambda関数
- 登録データを保存するDynamoDB

このアプローチにより、堅牢な認証・認可のためにAWS Cognitoを活用しながら、MCP仕様のDCR推奨事項への準拠を維持できます。

**セキュリティノート**: この実装では追加認証なしの匿名DCRを使用しています。本番環境では以下の追加を検討してください：
- 登録リクエストのレート制限
- クライアント認証（mTLS、初期アクセストークン）
- 新しいクライアントの承認ワークフロー
- 動的登録されたクライアントの制限されたスコープアクセス

登録プロセスのセキュリティを強化するため、[DCRセキュリティ推奨事項](./dcr-security-recommendations.md)を参照してください。

## クイックスタート

### 前提条件
- Node.jsがインストールされていること
- 以下へのアクセス権限を持つAWSテストアカウント：
    - 認証サーバー用Cognito（1ユーザープール、2アプリクライアント）
    - DCR用API Gateway / Lambda / DynamoDB（2リソース、2関数、1テーブル）
    - デプロイ用CloudFormation（1スタック）
- OAuth 2.1フローの基本知識

### セットアップ
1. リポジトリのクローン
   ```bash
   git clone https://github.com/empires-security/mcp-oauth2-aws-cognito.git
   cd mcp-oauth2-aws-cognito
   ```

2. クライアントとサーバーの依存関係をインストール
   ```bash
   npm run install:all
   ```

3. AWSリソースをデプロイ
   ```bash
   npm run deploy
   ```

4. 生成された`.env`ファイルを確認：
   - `src/client/.env`
   - `src/auto-client/.env`
   - `src/mcp-server/.env`
   - `.env.example`ファイルと比較
   - 必要に応じてCLIENT_SECRETを手動で確認/更新

### アプリケーションの実行
1. クライアントとサーバーを同時に起動
   ```bash
   npm run dev
   ```
2. http://localhost:3000 にアクセスしてOAuthフローをテスト

3. 新しいユーザーの登録
   - "Log in"ボタンをクリック
   - CognitoホストUIで"Sign up"を選択
   - 新しいユーザーアカウントを作成
   - メールに送信された確認コードを入力してアカウントを確認
   - 確認が成功すると、アプリケーションにリダイレクトされます

4. "Fetch MCP Data"ボタンをクリックして、MCPサーバーへの認証済みリクエストを作成

5. http://localhost:3002 にアクセスして、DCRフロー、Dynamic Client Registration機能付き自動発見クライアントをテスト

### クリーンアップ
1. AWSリソースをクリーンアップ
   ```bash
   npm run cleanup
   ```

詳細なセットアップ手順については、[セットアップガイド](./setup-guide.md)を参照してください。

## 貢献

貢献を歓迎します！お気軽にPull Requestを提出してください。

1. リポジトリをフォーク
2. 機能ブランチを作成 (`git checkout -b feature/amazing-feature`)
3. 変更をコミット (`git commit -m 'Add some amazing feature'`)
4. ブランチにプッシュ (`git push origin feature/amazing-feature`)
5. Pull Requestを開く

## 参考資料

- [Model Context Protocol 公式仕様](https://modelcontextprotocol.io/specification/draft/basic/authorization)
- [OAuth 2.1 ドラフト](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-12)
- [OAuth 2.0 Protected Resource Metadata](https://datatracker.ietf.org/doc/rfc9728/)
- [OAuth 2.0 Dynamic Client Registration Protocol](https://datatracker.ietf.org/doc/rfc7591/)
- [AWS Cognito デベロッパーガイド](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html)

## ライセンス

このプロジェクトはMITライセンスの下でライセンスされています - 詳細はLICENSEファイルを参照してください。

## 作者

- **Empires Security Labs** 🚀
