# 日本語ドキュメント

このフォルダには、MCP + OAuth2.1 + AWS Cognitoプロジェクトの日本語版ドキュメントが含まれています。

## 📋 ドキュメント一覧

### 主要ドキュメント
- **[README.md](./README.md)** - プロジェクト概要と基本情報
- **[setup-guide.md](./setup-guide.md)** - 詳細なセットアップ手順
- **[architecture-guide.md](./architecture-guide.md)** - アーキテクチャと設計の詳細説明

### セキュリティドキュメント
- **[dcr-security-recommendations.md](./dcr-security-recommendations.md)** - Dynamic Client Registrationのセキュリティ推奨事項

### 図表・ダイアグラム
- **[mcp-oauth-architecture.mermaid](./mcp-oauth-architecture.mermaid)** - システムアーキテクチャ図
- **[mcp-oauth-sequence.mermaid](./mcp-oauth-sequence.mermaid)** - OAuth認証シーケンス図
- **[mcp-oauth-sequence-dcr.mermaid](./mcp-oauth-sequence-dcr.mermaid)** - Dynamic Client Registrationシーケンス図

## 🚀 クイックスタート

このプロジェクトを始めるには：

1. **[README.md](./README.md)** でプロジェクトの概要を理解
2. **[setup-guide.md](./setup-guide.md)** に従って環境をセットアップ
3. **[architecture-guide.md](./architecture-guide.md)** でアーキテクチャの詳細を学習

## 🏗️ プロジェクト構成

```
MCP OAuth2.1 + AWS Cognito
├── MCPサーバー (リソースサーバー)
├── 静的クライアント
├── 自動発見クライアント (DCR対応)
└── AWS Cognito (認証サーバー)
```

## 🔐 セキュリティ

本番環境でのデプロイを検討している場合は、**[dcr-security-recommendations.md](./dcr-security-recommendations.md)** を必ずお読みください。

## 📞 サポート

技術的な質問やサポートが必要な場合は、プロジェクトのメインリポジトリでIssueを作成してください。
