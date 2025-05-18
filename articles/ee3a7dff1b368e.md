---
title: "Dockerコマンド総合ガイド"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Docker]
published: true
published_at: 2025-05-05 17:00
---

## はじめに

- 本記事では、学習したDockerコマンドを体系的にまとめております。
- Dockerの概念や詳細な解説は本記事の範囲外とし、実用的なコマンドリファレンスとしてご活用いただけるよう構成しております。
- docker-composeについては、内容の明瞭さを保つため、別の記事にて詳しくご紹介させていただく予定です。

## 基本的なコマンド構造

Dockerコマンドは基本的に以下の構造で実行します：

```bash
docker 対象 操作
```

## コンテナの起動と作成

### コンテナを新規作成し起動する方法

```bash
docker run --name <コンテナ名> -p <ホスト側のポート>:<コンテナ側のポート> -d <イメージ>
```

V2形式での実行方法：

```bash
docker container run --name <コンテナ名> -p <ホスト側のポート>:<コンテナ側のポート> -d <イメージ>
```

以下は実際の使用例です。  
`docker-nginx`という名前のコンテナを作成し、バックグラウンドで実行しています。  
この例ではホストの80番ポートをコンテナの80番ポートにマッピング（ポートフォワーディング）しています。

```bash
docker run --name docker-nginx -p 80:80 -d nginx
```

| オプション | 機能説明                                                   |
| :--------- | :--------------------------------------------------------- |
| --name     | コンテナに識別しやすい名前を付与します                     |
| -p         | ホスト側とコンテナ側のポート番号をマッピングします         |
| -d         | コンテナをバックグラウンドで実行します（デタッチドモード） |

### 一時的なコンテナを起動する方法（終了時に自動削除）

```bash
docker run --rm -p 80:80 [イメージID]
```

| オプション | 機能説明                                           |
| :--------- | :------------------------------------------------- |
| --rm       | コンテナ終了時に自動的にコンテナを削除します       |
| -p         | ホスト側とコンテナ側のポート番号をマッピングします |

### Dockerイメージのビルド方法

```bash
docker build -t [イメージ名]:[タグ] [Dockerfileのパス]
```

V2形式での実行方法：

```bash
docker image build -t [イメージ名]:[タグ] [Dockerfileのパス]
```

## 状態確認コマンド

### イメージ一覧の表示方法

```bash
docker images
```

V2形式での実行方法：

```bash
docker image ls
```

### コンテナ一覧の表示方法

```bash
docker ps -a
```

V2形式での実行方法：

```bash
docker container ls -a
```

### コンテナの詳細情報確認方法

```bash
docker inspect [コンテナID]
```

V2形式での実行方法：

```bash
docker container inspect [コンテナID]
```

### コンテナのログ確認方法

```bash
docker logs [コンテナID]
```

V2形式での実行方法：

```bash
docker container logs [コンテナID]
```

### コンテナのリソース使用状況確認方法

```bash
docker stats
```

V2形式での実行方法：

```bash
docker container stats
```

### システム情報の確認方法

```bash
docker info
```

## コンテナへの接続方法

### コンテナへの接続（docker-compose使用時）

```bash
docker-compose exec [コンテナ名] bash
```

### コンテナへの接続（dockerコマンド使用時）

```bash
docker exec -it [コンテナID] bash
```

V2形式での実行方法：

```bash
docker container exec -it [コンテナID] bash
```

## コンテナの停止方法

### 特定のコンテナを停止する方法

```bash
docker stop [コンテナID]
```

V2形式での実行方法：

```bash
docker container stop [コンテナID]
```

### すべてのコンテナを一括停止する方法

```bash
docker stop $(docker ps -a -q)
```

V2形式での実行方法：

```bash
docker container stop $(docker container ls -a -q)
```

## 削除関連コマンド

### Dockerイメージの削除方法

```bash
docker rmi [イメージID] [イメージID]...
```

V2形式での実行方法：

```bash
docker image rm [イメージID] [イメージID]...
```

### コンテナの削除方法

```bash
docker rm [コンテナID]
```

V2形式での実行方法：

```bash
docker container rm [コンテナID]
```

#### 削除時にエラーが発生した場合の対処法

強制削除オプション（-f）を使用することで、実行中のコンテナでも確実に削除することができます。

```bash
docker rm -f [コンテナID]
```

V2形式での実行方法：

```bash
docker container rm -f [コンテナID]
```

### すべてのコンテナを一括削除する方法

```bash
docker rm $(docker ps -a -q)
```

V2形式での実行方法：

```bash
docker container rm $(docker container ls -a -q)
```

### 未使用リソースの一括削除

#### 未使用イメージの削除

```bash
docker image prune -a
```

#### 未使用ボリュームの削除

```bash
docker volume prune
```

#### 未使用ネットワークの削除

```bash
docker network prune
```

#### システム全体のクリーンアップ（総合的な不要リソース削除）

```bash
docker system prune -a
```

## ネットワーク関連コマンド

### ネットワーク一覧の表示方法

```bash
docker network ls
```

### ネットワークの新規作成方法

```bash
docker network create [ネットワーク名]
```

## ボリューム関連コマンド

### ボリューム一覧の表示方法

```bash
docker volume ls
```

### ボリュームの新規作成方法

```bash
docker volume create [ボリューム名]
```

## ファイル転送コマンド

### コンテナからホストへのファイルコピー方法

```bash
docker cp [コンテナID]:[コンテナ内のパス] [ホスト側のパス]
```

### ホストからコンテナへのファイルコピー方法

```bash
docker cp [ホスト側のパス] [コンテナID]:[コンテナ内のパス]
```

## Docker Hub操作コマンド

### Docker Hubへのログイン方法

```bash
docker login
```

### Docker Hubへのイメージ公開（プッシュ）方法

```bash
docker push [ユーザー名]/[リポジトリ名]:[タグ]
```

V2形式での実行方法：

```bash
docker image push [ユーザー名]/[リポジトリ名]:[タグ]
```
