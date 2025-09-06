---
title: "公式推奨のDockerfileの記述法とWebアプリ作成時のDockerfile作成例"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Docker", "Dockerfile"]
published: true
published_at: 2025-05-06 09:00
---

## はじめに

[こちらの記事](https://zenn.dev/wan0ri/articles/ee3a7dff1b368e)では、Dockerコマンドを使って起動・停止・状態確認・削除などのコマンドを掲載していますが、いちいちコマンドを入力してDockerを扱うのは非常に面倒になってきます。

そこで、Dockerfileを使用してDockerの取り扱いを楽にしていきましょう。

## 概要

### そもそもDockerfileとは？

- 特定の命令セットを記述することで、コンテナのビルド手順を定義するファイル
- 主に開発者が使用し、プロジェクトの依存関係や必要な設定を一元管理する
- これにより、チーム全体で一貫した環境を保つことができ、環境設定の手間を大幅に削減できる

### Dockerfileが重要な理由

Dockerfileは、コンテナの一貫性と再現性を保証します。  
これにより、開発環境と本番環境の差異を減らし、デプロイの問題を軽減することができます。  
Dockerfileを使用することで、環境構築が自動化され、人的ミスのリスクが減少します。

### Dockerfileの基本構成

Dockerfileの基本構成は、ベースイメージの指定から始まり、必要なファイルのコピー、パッケージのインストール、環境変数の設定などが含まれています。  
これらの命令は、一つのファイルに順序立てて記述され、Dockerビルドプロセスで順に実行されます。

### DockerfileとDockerイメージの関係

Dockerfileは、Dockerイメージを生成するための設計図です。  
Dockerfileに記述された命令に従って、Dockerイメージがビルドされ、そのイメージからコンテナが起動されます。  
イメージは、各ステップのスナップショットとして保存され、効率的な再利用が可能です。

### Dockerfileの利用シーン

Dockerfileは、開発環境の構築やテストの自動化、CI/CDパイプラインの一部として使用されます。  
また、異なる環境間での移植性が高いため、チーム全体で統一された開発環境を提供することができます。  
これにより、開発効率と品質が向上します。

## 記述方法

### 基本構文

Dockerfileは命令ベースで記述します。  
代表的なものは以下の通りです：

- `FROM`: ベースイメージ指定
- `RUN`: コマンド実行（イメージ作成時）
- `COPY / ADD`: ファイルやディレクトリをコピー
- `WORKDIR`: 作業ディレクトリ設定
- `CMD`: コンテナ起動時のデフォルトコマンド
- `ENTRYPOINT`: コンテナ起動時の実行バイナリ指定
- `ENV`: 環境変数定義
- `EXPOSE`: ポート公開
- `VOLUME`: ボリューム定義

### ベストプラクティス

[Docker公式のベストプラクティス](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)に沿うとよいと思われます。  
(企業様や使用者によりベストプラクティスが異なると思いますが、了承願いたいです。あくまで公式ベースということで…)

公式を読み取ると、以下の内容でまとめられています：

- 命令の順序についての推奨事項
- イメージのキャッシュ利用とビルド効率
- イメージサイズを小さくする方法
- レイヤーの最適化
- セキュリティの考慮

これを考慮すると、順番としては以下に収まっていきます：

- `FROM`: ベースイメージ指定
- `LABEL`: バージョン、作者などのメタ情報
- `ENV`: 環境変数定義
- `WORKDIR`: 作業ディレクトリ設定
- `COPY / ADD`: ファイルやディレクトリをコピー
- `RUN`: コマンド実行（イメージ作成時）
- `EXPOSE`: ポート公開
- `VOLUME`: ボリューム定義
- `ENTRYPOINT / CMD`: コンテナ起動時の実行バイナリ指定、コンテナ起動時のデフォルトコマンド

💡キャッシュを活用するため、変更頻度が少ないものほど先に書くとよいです。

### サイズを小さくするコツ

- 軽量ベースイメージを使う：
  - 例: alpine, debian:slim
- 不要なファイルをCOPYしない：
  - `.dockerignore`を活用
- マルチステージビルド：
  - ビルド専用と実行専用で**イメージを分ける**
- RUNをまとめる：
  - レイヤーを減らすために`&&`で結合
- キャッシュ削除：
  - `apk add ... && rm -rf /var/cache/apk/\*` ※Alpineの場合

## 記述例

### 構成パターン

1. Reactアプリ（静的ファイルをビルド）
   通常、npm run buildでbuild/ディレクトリが作られ、それをNginxなどで配信するかRailsのpublic/フォルダに置く。
2. Railsアプリ
   必要最小限のライブラリだけインストールし、開発用パッケージやキャッシュは削除。

### Dockerfileを分ける or 1つにまとめる？

- **開発時は分けた方が良い** （Docker Composeなどでフロント・バックを別コンテナにする）。
- 本番では、Railsが静的ファイルを配信するなら1つのイメージにまとめてもOKです。

ここでは 1つにまとめる場合の例を記載します。

### 実践的なDockerfile例（マルチステージ）

:::details Dockerfileを1つにまとめた例

```Dockerfile
# ===== フロントエンドビルドステージ =====
FROM node:20-alpine AS frontend-build

WORKDIR /app/frontend

# package.jsonだけ先にCOPYしてキャッシュ活用
COPY frontend/package*.json ./

RUN npm ci

COPY frontend/ ./

RUN npm run build


# ===== バックエンドビルドステージ =====
FROM ruby:3.2-alpine AS backend-build

# OSパッケージインストール（例: PostgreSQL対応の場合）
RUN apk add --no-cache build-base postgresql-dev nodejs yarn

WORKDIR /app

# Gemfileだけ先にCOPYしてキャッシュ活用
COPY backend/Gemfile backend/Gemfile.lock ./

# bundle install（--without development testで容量削減）
RUN bundle install --without development test

# アプリケーション全体をCOPY
COPY backend/ ./

# ===== 本番実行ステージ =====
FROM ruby:3.2-alpine

# OSパッケージ（最小限）
RUN apk add --no-cache postgresql-client

WORKDIR /app

# バンドル済みgemsをCOPY
COPY --from=backend-build /usr/local/bundle /usr/local/bundle

# アプリをCOPY
COPY --from=backend-build /app /app

# フロントエンドビルド成果物をpublic/に配置
COPY --from=frontend-build /app/frontend/build /app/public

# 環境変数やポート
ENV RAILS_ENV=production
EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]
```

:::

### .dockerignore（容量削減の大事なポイント）

RailsとReactそれぞれに必須：

:::details .dockerignore

```.dockerignore
# 共通
node_modules
log
tmp
.git
.dockerignore

# Rails特有
vendor/bundle
coverage
public/assets
public/packs

# React特有
build
```

:::

### .dockerignoreの活用方法

`.dockerignore`ファイルは、Dockerイメージをビルドする際に無視すべきファイルやディレクトリを指定するためのもの。  
このファイルをプロジェクトのルートディレクトリ（Dockerfileと同じ場所）に配置することで、以下のメリットが得られる。

1. **ビルド時間の短縮**：不要なファイルをCOPYしないため、ビルドプロセスが高速化されます。

2. **イメージサイズの削減**：
   `node_modules`や一時ファイルなどの大きなディレクトリを除外することで、最終イメージのサイズを大幅に削減できます。

3. **セキュリティの向上**：
   `.env`ファイルや秘密鍵などの機密情報を誤ってイメージに含めることを防ぎます。

使用方法は非常に簡単で、`.gitignore`と同様のパターンマッチング構文を使用する。  
Dockerビルドコマンドを実行すると、自動的に`.dockerignore`ファイルが読み込まれ、指定されたパターンに一致するファイルは`COPY`や`ADD`命令で無視されます。

### Dockerfileの意識ポイント

1. マルチステージで最終イメージにはフロントの `build` だけが入ります。
2. COPY順序を意識してキャッシュが効きやすくします。
3. node_modulesやvendor/bundleは **.dockerignore** で除外します。
4. 不要な開発ツールは最終イメージに含めません：

- apk addなども最終段階では最小限。
- bundle install --without development testでサイズ最適化。

## 補足

- **Nginx併用**：
  RailsではなくNginxを使ってフロントエンドを配信したい場合は、Nginx用Dockerfileも作成してReactのbuildディレクトリを/usr/share/nginx/htmlに置くのが一般的です。
- **Docker Compose併用**：
  フロント・バックを別々に動かすならdocker-compose.ymlでサービスを分けて管理したほうが、開発中は便利です。
