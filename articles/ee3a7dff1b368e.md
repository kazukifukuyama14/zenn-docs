---
title: "Dockerコマンドまとめ"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Docker]
published: true
published_at: 2025-05-05 17:00
---

## はじめに

- 学習したDockerコマンドのまとめを下記に記載する。
- Dockerに関する詳細は記載しない。（あくまでコマンドのまとめだけ実施）
- docker-composeは記事の内容が煩雑になると思うので別途まとめ実施

## 基本操作

```bash
docker 対象 操作
```

## 起動

### コンテナを作成し、起動

```bash
docker run --name <コンテナ名> -p <ホスト側のポート>:<コンテナ側のポート> -d <イメージ>
```

下記は`docker-nginx`という名前のコンテナを作成し、バックグラウンドで実行しているコマンド。
ホストの80番ポートをコンテナの80番ポートにマッピング（ポートフォワーディング）している。

```bash
docker run --name docker-nginx -p 80:80 -d nginx
```

| オプション |                       役割 |
| :--------- | -------------------------: |
| --name     |     コンテナに名前をつける |
| -p         |       ポート番号を指定する |
| -d         | バックグラウンドで実行する |

### プロセスを停止したらコンテナを破棄するモードでコンテナを起動

```bash
docker run --rm -p 80:80 [イメージID]
```

| オプション |                                 役割 |
| :--------- | -----------------------------------: |
| --rm       | コンテナ終了時にコンテナ自動的に削除 |
| -p         |                 ポート番号を指定する |

## 状態確認

### イメージの一覧表示

```bash
docker images
```

### 起動中の docker コンテナの一覧を表示

```bash
docker ps -a
```

### 起動中のコンテナの詳細情報を確認

```bash
docker inspect [コンテナID]
```

## 接続

### docker コンテナに接続

```bash
docker-compose exec [コンテナにつけた名前] bash
```

</br>

## 停止

### コンテナプロセスを停止

```bash
docker stop [コンテナID]
```

### 全てのコンテナプロセスを停止

```bash
docker stop $(docker ps -a -q)
```

## 削除

### docker イメージを削除

```bash
docker rmi [イメージID] [イメージID]...
```

### コンテナを削除

```bash
docker rm [コンテナID]
```

#### エラーが出たら

オプションで-fを付ける事で確実に削除する事が可能。

```bash
docker rm -f [コンテナID]
```

### 全てのコンテナを削除

```bash
docker rm $(docker ps -a -q)
```

### 使用していないイメージ・コンテナをすべて消去

```bash
docker image prune -a
```

追記分あれば適宜更新予定。
