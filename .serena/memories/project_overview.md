# MCP OAuth2.1 + AWS Cognito プロジェクト概要

## プロジェクトの目的
このプロジェクトは、Model Context Protocol (MCP) サーバーをOAuth 2.1認証フローで保護する方法を実演するデモンストレーションです。AWS Cognitoを認証サーバーとして使用していますが、**プロバイダーに依存しない**設計で、任意のOAuth 2.1準拠の認証サーバーと連携可能です。

## 主な特徴
- MCP Authorization Specification (2025-06-18版) に基づく実装
- OAuth 2.1 Authorization Code Flow with PKCE
- 動的クライアント登録 (DCR) サポート
- Protected Resource Metadata (PRM) ドキュメント発見
- 完全に動的な認証サーバーメタデータ発見
- プロバイダーに依存しないトークン検証

## アーキテクチャ
1. **MCP Server** (Resource Server): Express.jsで実装されたリソースサーバー
2. **Client**: 静的な設定済みクレデンシャルを使用するクライアント
3. **Auto-Discovery Client**: 動的登録をサポートする自動発見クライアント
4. **Authorization Server**: AWS Cognito（他のOAuth 2.1準拠サーバーに置き換え可能）

## 技術スタック
- **言語**: Node.js (JavaScript)
- **フレームワーク**: Express.js
- **認証**: OAuth 2.1 + PKCE
- **インフラ**: AWS Cognito, CloudFormation
- **主要依存関係**: axios, jsonwebtoken, jwk-to-pem, cors, express-session