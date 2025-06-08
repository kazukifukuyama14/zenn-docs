---
title: "【完全版】React + Node.js + PostgreSQLでKubernetesを実践学習！フルスタックアプリ構築チュートリアル"
emoji: "🚀"
type: "tech"
topics: ["kubernetes", "react", "nodejs", "postgresql", "docker"]
published: true
published_at: 2026-06-08 21:00
---

# 概要

Kubernetesを学習したいけれど、「Hello World」レベルのサンプルでは物足りない...そんな方に向けた実践的なチュートリアルです。

本記事では、**React（フロントエンド）+ Node.js/Express（バックエンド）+ PostgreSQL（データベース）** を組み合わせた本格的なフルスタックWebアプリケーションを、Kubernetesクラスター上に構築する手順を詳しく解説します。

## 🎯 この記事で学べること

- Kubernetesの基本概念（Pod、Deployment、Service、ConfigMap、PersistentVolume）
- マイクロサービス間の通信設定
- Nginxを使ったリバースプロキシ設定
- データベースの永続化
- 実際の開発で遭遇するトラブルシューティング

## 🏗️ 構築するアプリケーション

シンプルなユーザー管理システムを構築します：

- **フロントエンド**: Reactでユーザー一覧を表示
- **バックエンド**: Express APIでユーザーデータを提供
- **データベース**: PostgreSQLでユーザー情報を永続化

![アプリケーション完成イメージ](/images/kubernetes-app/infra.png)

## 📁 GitHubリポジトリ

完全なソースコードはこちらで公開しています：

**🔗 [kubernetes-webapp-tutorial](https://github.com/kazukifukuyama14/kubernetes-webapp-tutorial)**

```bash
git clone https://github.com/kazukifukuyama14/kubernetes-webapp-tutorial.git
```

## 📋 前提条件

### 必要なツール

以下のツールが事前にインストールされている必要があります：

| ツール             | バージョン | 用途                   |
| ------------------ | ---------- | ---------------------- |
| **Docker Desktop** | 最新版     | コンテナ実行環境       |
| **Minikube**       | v1.25+     | ローカルKubernetes環境 |
| **kubectl**        | 最新版     | Kubernetesクライアント |
| **Git**            | 最新版     | ソースコード管理       |

### インストール確認

```bash
# バージョン確認コマンド
docker --version
minikube version
kubectl version --client
git --version
```

### 推奨スペック

- **メモリ**: 8GB以上
- **CPU**: 4コア以上
- **ディスク容量**: 10GB以上の空き容量

### 参考:著者のスペック

![macbookspec](/images/kubernetes-app/macbookspec.png)

:::message
**初心者の方へ**
KubernetesやDockerの基本概念について不安な方は、まず公式ドキュメントの入門セクションを一読することをお勧めします。  
ただし、本記事では実際の手順を詳しく解説するため、基本的な知識がなくても進められるよう配慮しています。
:::

## 🏛️ アーキテクチャ概要

今回構築するシステムの全体像：

```bash
[ブラウザ]
    ↓ HTTP
[Minikube Service]
    ↓
[Frontend Pod (React + Nginx)]
    ↓ /api/* → プロキシ
[Backend Pod (Node.js/Express)]
    ↓ SQL
[PostgreSQL Pod]
    ↓
[Persistent Volume]
```

それでは、実際に構築を始めていきましょう！🚀

