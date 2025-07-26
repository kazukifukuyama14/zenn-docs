---
title: "WAS × Semeru Runtime 組み込み検証まとめ"
emoji: "🗃️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS, ECR, IAM, Docker, Dockerfile]
published: true
---

## 検証目的

IBM公式の WebSphere Application Server（WAS）コンテナに、Semeru Runtime Certified Edition(OpenJDK 21.0.7+6)を組み込み、ECRにプッシュできるかを検証しました。

### 経緯

客先環境でSemeru RuntimeをGitで管理する想定でいましたが、容量が大きすぎて管理することが難しい状況になってしまいました。  
Git Large File Storageで管理することも検討しましたが、お客様より以下提案がありました。

```txt
今後多数のアプリをデプロイしないといけないため、専用リポジトリで管理することは可能か？
```

そのような実績が常駐先で無かったため、PoC環境にて検証することになりました。

## 構成概要

- ベースイメージ①：Semeru Runtime（ibm-semeru-certified-jdk_x64_linux_21.0.7.0.tar）
- ベースイメージ②：IBM公式 WAS（ibmcom/websphere-traditional:latest）
- 組み込み方法：Semeru Runtime を `/opt/java/jdk-21.0.7+6` に配置し、環境変数 JAVA_HOME と PATH を設定

https://developer.ibm.com/languages/java/semeru-runtimes/downloads/?license=IBM

## IAMポリシー 事前準備

今回はECRに対してリポジトリをプッシュ・プルしますので、以下の権限を実行ユーザーのIAMポリシーに組み込む必要があります。

- `ecr:GetAuthorizationToken` (ログイン用)
- `ecr:BatchCheckLayerAvailability` (プル時に必要)
- `ecr:GetDownloadUrlForLayer` (プル時に必要)
- `ecr:BatchGetImage` (プル時に必要)
- `ecr:InitiateLayerUpload` 、 `ecr:UploadLayerPart、ecr:CompleteLayerUpload` (プッシュ時に必要)
- `ecr:PutImage` (プッシュ時に必要)
- `ecr:ListImages` (リスト取得)
- `ecr:GetDownloadUrlForLayer` (プル時に必要)

以下はIAMポリシーの例になります。  
※ARNについてはポリシーに含めたいものを指定してください。

```json:IAMポリシー
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:CompleteLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:PutImage",
                "ecr:ListImages",
                "ecr:BatchGetImage",
                "ecr:GetAuthorizationToken",
                "ecr:GetDownloadUrlForLayer"
            ],
            "Resource": "arn:aws:ecr:ap-northeast-1:[アカウント番号]:repository/○○*"
        },
        {
            "Effect": "Allow",
            "Action": "ecr:GetAuthorizationToken",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ecs:*"
            ],
            "Resource": [
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:cluster/○○-*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:service/○○-*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:task/○○-*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:cluster/○○*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:service/○○*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:task/○○*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ecs:ListTasks"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole",
                "iam:CreateRole"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ssm:StartSession"
            ],
            "Resource": [
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:cluster/○○-*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:service/○○-*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:task/○○-*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:cluster/○○*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:service/○○*",
                "arn:aws:ecs:ap-northeast-1:[アカウント番号]:task/○○*"
            ]
        }
    ]
}
```

## 各種ファイル構成

### Dockerfile

```Dockerfile:Semeru Runtimeリポジトリ用Dockerfile
# ベースイメージとして軽量なAlpine Linuxを使用
FROM alpine:latest

# 作業ディレクトリを /opt/java に設定
WORKDIR /opt/java

# Semeru JDK の tar ファイルをコンテナにコピー
COPY ibm-semeru-certified-jdk_x64_linux_21.0.7.0.tar .

# tar ファイルを展開して JDK をインストール
RUN tar -xf ibm-semeru-certified-jdk_x64_linux_21.0.7.0.tar

# JAVA_HOME 環境変数を設定（JDKのインストール先を指定）
ENV JAVA_HOME=/opt/java/jdk-21.0.7+6

# PATH 環境変数に JDK の bin ディレクトリを追加
ENV PATH="$JAVA_HOME/bin:$PATH"
```

```Dockerfile:WAS組み込み用Dockerfile
# Stage 1: Semeru Runtime を取得
FROM [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com/xanet-semeru-runtime:21.0.7.0 AS semeru

# Stage 2: IBM公式WASイメージ
FROM ibmcom/websphere-traditional:latest AS was

# Semeru Runtime を WAS に組み込み
COPY --from=semeru /opt/java /opt/java

# 作業ディレクトリ
WORKDIR /tmp

# WASイメージのユーザーを確認（wasユーザーが存在する前提）
USER was

# テスト用のシェルをコピー（wasユーザーが書き込み可能な場所に）
COPY --chown=was:was test.sh /tmp/test.sh

# 環境変数の設定
ENV JAVA_HOME=/opt/java/jdk-21.0.7+6
ENV PATH="$JAVA_HOME/bin:$PATH"

# 起動コマンド
CMD ["bash", "/tmp/test.sh"]
```

### test.sh の役割

```bash:test.sh
#!/bin/bash
COUNT=100000
INTERVAL=5
for i in $(seq 1 $COUNT)
do
  echo "test"
  sleep $INTERVAL
done
```

- コンテナが即終了するのを防ぐための ダミーの長時間実行プロセス
- 5秒間隔で "test" を出力し続けることで、コンテナが安定して稼働するかを確認

## 閑話休題

私自身がJavaについて全く知識がないため、改めて今回の事例を調べてみました。

### Semeru Runtimesとは？

Semeru Runtimesは、**IBMが提供するOpenJDKベースのJavaランタイム環境**です。  
特にOpenJ9という軽量で高性能なJVM（Java Virtual Machine）を搭載しており、以下のような特徴があります：

- メモリ使用量が少ない
- 起動時間が速い
- パフォーマンスが安定している

### WASにSemeru Runtimesを組み込む目的

WASはJava EEベースのアプリケーションサーバであり、Javaランタイム環境（JRE/JDK）に依存しています。  
つまり、WASが動作するにはJVMが必要です。  
Semeru RuntimesをWASに組み込むことで、以下のようなメリットが得られます：

| 目的                  | 説明                                                                          |
| --------------------- | ----------------------------------------------------------------------------- |
| パフォーマンス向上    | OpenJ9の特性により、WASの起動時間やメモリ使用量が改善される可能性があります。 |
| 軽量化                | コンテナ環境でのリソース効率が向上し、よりスリムなイメージが作成可能です。    |
| IBMサポートとの整合性 | IBM製品同士の互換性が高く、サポート面でも有利です。                           |
| セキュリティと安定性  | IBMが提供するランタイムなので、セキュリティパッチや更新が信頼できます。       |

### なぜ「組み込む」必要があるのか？

WASのDockerイメージにSemeru Runtimesを組み込むことで、WASが依存するJVMのバージョンや種類を明示的に管理できるようになります。  
これは、**運用面での安定性やトラブルシューティングの効率化にもつながります。**

## 手順

作業を進める前に、事前にECRへ空のリポジトリを作成しておきます。

- Semeru Runtimes：semeru-runtime
- WAS組み込み：was-semeru

双方ともに「タグのイミュータビリティ」は**ミュータブル**、「暗号化タイプ」は**AES-256**で作成しております。

手順としては、以下の手順で進めていきます。

1. Semeru RuntimesをECRへプッシュ
2. WASにSemeru Runtimesを組み込んだものをECRへプッシュ
3. 組み込んだコンテナに接続して事後確認

### Semeru Runtimes

#### イメージビルド

```bash:docker buildコマンド
docker build -t semeru-runtime:21.0.7.0 .
```

```bash:docker buildコマンド実行ログ
[+] Building 7.8s (9/9) FINISHED                                                                         docker:default
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 266B                                                                               0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                   1.4s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [1/4] FROM docker.io/library/alpine:latest@sha256:4bcff63911fcb4448bd4fdacec207030997caf25e9bea4045fa6c8c44de  0.0s
 => [internal] load build context                                                                                  3.8s
 => => transferring context: 414.94MB                                                                              3.8s
 => CACHED [2/4] WORKDIR /opt/java                                                                                 0.0s
 => [3/4] COPY ibm-semeru-certified-jdk_x64_linux_21.0.7.0.tar .                                                   0.9s
 => [4/4] RUN tar -xf ibm-semeru-certified-jdk_x64_linux_21.0.7.0.tar                                              1.0s
 => exporting to image                                                                                             0.7s
 => => exporting layers                                                                                            0.7s
 => => writing image sha256:674748734c43241d5d833794c92c4219e232d7b4b12dc9437e21053098170495                       0.0s
 => => naming to docker.io/library/xanet-semeru-runtime:21.0.7.0                                                   0.0s
```

※タグは0.0.1の方がわかりやすいとは思いますが、Semeru Runtimesのバージョンが `21.0.7.0` ですので意図的にこのバージョンにしています。

#### Semeru Runtimeリポジトリ(tarファイル)ECRプッシュ対応

```bash
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com

docker tag semeru-runtime:21.0.7.0 [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com/semeru-runtime:21.0.7.0

docker image ls

docker push [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com/semeru-runtime:21.0.7.0
```

AWSのマネジメントコンソールにて、リポジトリがプッシュできていたら一旦は完了です。

### WAS組み込み

#### ビルド

```bash:docker buildコマンド
docker build -t was-semeru:0.0.1 .
```

```bash:docker buildコマンド実行ログ
[+] Building 5.7s (11/11) FINISHED                                                                       docker:default
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 776B                                                                               0.0s
 => [internal] load metadata for docker.io/ibmcom/websphere-traditional:latest                                     1.5s
 => [internal] load metadata for [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com/xanet-semeru-runtime:21.0.7.0   0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [semeru 1/1] FROM [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com/xanet-semeru-runtime:21.0.7.0              0.0s
 => [internal] load build context                                                                                  0.0s
 => => transferring context: 153B                                                                                  0.0s
 => CACHED [was 1/4] FROM docker.io/ibmcom/websphere-traditional:latest@sha256:ae89e05d980537c9721b81ea392b48a975  0.0s
 => [was 2/4] COPY --from=semeru /opt/java /opt/java                                                               1.4s
 => [was 3/4] WORKDIR /tmp                                                                                         0.0s
 => [was 4/4] COPY --chown=was:was test.sh /tmp/test.sh                                                            0.0s
 => exporting to image                                                                                             1.3s
 => => exporting layers                                                                                            1.2s
 => => writing image sha256:6052eca0d584c9953612f8927d3dec3acf562126798eedde13e95111e8e07178                       0.0s
 => => naming to docker.io/library/xanet-was-semeru:0.0.1                                                          0.0s
```

#### WAS組み込みリポジトリ ECRプッシュ対応

```bash
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com

docker tag was-semeru:0.0.1 [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com/was-semeru:0.0.1

docker image ls

docker push [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com/was-semeru:0.0.1
```

### WAS組み込みコンテナ起動、動作確認

#### イメージ起動

```bash
docker run -d [アカウント番号].dkr.ecr.ap-northeast-1.amazonaws.com/was-semeru:0.0.1
```

#### コンテナ接続

```bash
docker container ls -a

docker exec -it [コンテナID] /bin/bash
```

#### 動作確認

```bash:実行コマンド
echo $JAVA_HOME
```

```bash:実行結果
/opt/java/jdk-21.0.7+6
```

---

```bash:実行コマンド
ls -l /opt/java/jdk-21.0.7+6
```

```bash:実行結果
total 36
-rw-r--r--  1 root root  788 May  4 17:19 README.txt
drwxr-xr-x  2 root root 4096 Jul 25 02:46 bin
drwxr-xr-x  5 root root 4096 Jul 25 02:46 conf
drwxr-xr-x  3 root root 4096 Jul 25 02:46 include
drwxr-xr-x  2 root root 4096 Jul 25 02:46 jmods
drwxr-xr-x 77 root root 4096 Jul 25 02:46 legal
drwxr-xr-x  9 root root 4096 Jul 25 02:46 lib
drwxr-xr-x  3 root root 4096 Jul 25 02:46 man
-rw-r--r--  1 root root 1747 May  4 17:21 release
```

---

```bash:実行コマンド
java --version
```

```bash:実行結果
java 21.0.7 2025-04-15 LTS
IBM Semeru Runtime Certified Edition 21.0.7.0 (build 21.0.7+6-LTS)
Eclipse OpenJ9 VM 21.0.7.0 (build openj9-0.51.0, JRE 21 Linux amd64-64-Bit Compressed References 20250415_442 (JIT enabled, AOT enabled)
OpenJ9   - 31cf5538b0
OMR      - 9bcff94a2
JCL      - 4da3a93510e based on jdk-21.0.7+6)
```

<br>

動作確認の結果より、以下が出力されたら確認としてはOKです。

- echoコマンドで、Semeru RuntimeのDockerfileで定義した `/opt/java/jdk-21.0.7+6` が出力されていること
- lsコマンドで、jdk-21.0.7+6配下にコンテナ資源が格納されていること
- java --versionで、 `java 21.0.7` が出力されていること

## あとがき

今回の出来事で、以下のことが学べました。

- **Semeru Runtimeは環境変数の設定だけでWASに組み込める**
  - `/opt/java/jdk-21.0.7+6` に配置し、JAVA_HOME と PATH を通すだけで動作可能。
- **IBM公式のWASイメージは非rootユーザーで動作する**
  - chmod などの操作には COPY --chown を使うなどの工夫が必要。
- **コンテナが即終了する問題にはダミースクリプトで対応可能**
  - test.sh のようなループスクリプトで、コンテナの動作確認がしやすくなる。
- **DockerfileのマルチステージビルドでSemeru Runtimeを効率的に組み込める**
  - イメージサイズや構成の最適化に有効。
- **WAS + Semeru Runtimeの組み合わせでも正常に動作することを確認**
  - 今後の本番環境適用やCI/CD連携の基盤として有望。
