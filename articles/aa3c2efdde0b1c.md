---
title: "ミリしらKubernetes〜ド初心者がKubernetesをある程度理解するまでの記録〜"
emoji: "⛴️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Kubernetes, Docker]
published: true
published_at: 2027-05-31 09:00
---

## 🏃 前提

筆者はDockerを業務で触ったことはございますが、Kubernetesを業務で触ったことがございません。  
そこで、  
**Kubernetesを1ミリもしらない状態でアウトプットしていけばどこまで理解することができるのか**  
をハンズオン形式で実践して理解を深めていきたいと思います。

### Docker

Dockerはコンテナを作成・管理するために必要です。  
以下のリンクから自分の環境に合わせてインストールします。
<https://docs.docker.com/engine/install/>

### IDE

Visual Studio Codeなど慣れているものでいいとは思いますが、私はCursorを使用していきます。
<https://www.cursor.com/ja>

また、テキストエディタには拡張機能「**Remote SSH**」が必要ですのでそちらをインストールしておきます。
<https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh>

### ターミナル

下記コマンドを入力し、インストールしていきます。

```bash
brew install kubernetes-cli
```

```bash
brew install
```

(参考)  
<https://qiita.com/Asaminnn/items/6d414d4776d964e94e96>

## 📝 概要

### Kubernetesとは？

「**Kubernetes**」とは、  
**コンテナ型仮想化技術を対象とした運用管理、自動化のツール、コンテナオーケストレーションツールの一つ**  
のことです。

### Dockerとの違い

コンテナと聞くとDockerを思い浮かべますが、DockerとKubernetesでは違いがあります。  
**Docker**は簡単にコンテナの作成や削除ができるソフトウェアツールです。  
必要なパッケージのコード化や環境の再配布、チーム開発時の環境の統一などに使用されます。

一方、**Kubernetes**は複数のコンテナを用いた開発に使用するソフトウェアツールです。  
各コンテナの状態を確認し、問題のあるコンテナを再起動する、といった管理が可能です。

この二つのツールの間では管理単位にも違いがあります。  
Dockerはある**マシン上の一つ一つのコンテナが最小の単位**で、コンテナの乗ったマシンであるノードも管理の単位です。

一方のKubernetesでは最小の単位は**Pod**です。  
Podは一つ以上のコンテナを持ちますが、1Podに1コンテナである場合が多いです。  
**1つ以上のPodを持つマシンがノード（ワーカーノード）** です。

より大きい単位としてはクラスタがあります。  
**クラスタはノードの集合体**で、1つ以上のワーカーノードとそれを管理するマスターノードを持ちます。

|          | Kubernetes            | Docker                       |
| -------- | --------------------- | ---------------------------- |
| **用途** | コンテナ管理          | アプリケーションのコンテナ化 |
| **単位** | Pod、ノード、クラスタ | コンテナ、ノード             |

### Kubernetesを使用するメリットとデメリット

Kubernetesは、非常に効率よく大規模なITインフラを運用・管理できるツールです。  
しかし、万能のツールというわけではなく、実際には以下のようなメリット・デメリットを持ちます。

#### メリット

- **コストを削減できる**
  Kubernetesには、負荷分散やリソース配分などを自動的に調整する機能が含まれています。

  こうした機能を活用することで、システムの安定稼働において非常に重要な「調整作業」を自動化できるため、運用コストの低減が可能です。

  また、過去の実績に基づいて効率よくリソースを使用できるため、従量課金制を採用するクラウドプラットフォームの月額利用料を節約することができます。

- **大量のコンテナを一括管理できる**
  Kubernetesを利用することで、大量のコンテナを容易に管理できます。

  例えば、多くのコンテナに対して設定を変更する際に、コンテナ1つ1つに変更作業を行うのは大変な労力がかかります。

  Kubernetesでは、複数のコンテナ間で設定ファイルを共有できるため、このような複数のコンテナに対する設定変更も個別に行う必要が無くなります。

  また、ローリングアップデートに対応しているため、デプロイ作業を手動で行う必要がないという点も大きなメリットです。

- **起動が高速・軽量で迅速なアプリケーション開発を実現できる**
  コンテナはアプリケーションが必要とするリソースのみが含まれているため、リソース消費を抑え、従来の仮想マシンよりも高速かつ軽量で動作します。

  これにより、リソースを効率的に活用でき、アプリケーション開発を迅速に進められます。

  市場の変化に対応し迅速なリリースが求められる昨今では、Kubernetesの活用は非常に重要です。

- **DevOpsを実現しやすい**
  DevOpsとは「開発と運用の一体化によって、システムを常に最新の状態に保ち、ユーザーにいち早く新しい価値を届ける」という考え方です。

  特にWebサービスなどの継続的な機能の更新とリリースを行うシステムで取り入れられています。

  Kubernetesには、アプリケーションの開発・運用に必要な機能がほぼ網羅されており、開発・運用をシームレスに連結することができます。

  本番環境を稼働させた状態で、改善点の実装と適用が行えるため、システム全体を常に最新・最善の状態に保ちつつダウンタイムを最小化することが可能です。これは「DevOps」の実現において威力を発揮します。

- **オンプレミスとクラウドのどちらでも利用できる**
  Kubernetesは複数のソフトウェア・ハードウェア上で動作します。

  更にクラウドのベンダーの多くがサポートしているためオンプレミス・クラウド問わずに利用可能です。

  使用用途や前提を問わず使用できるため、複数コンテナを用いた開発をする場合は積極的に使用したい技術と言えるでしょう。

- **アプリケーションのデプロイも容易になる**
  Kubernetesではコンテナを基にデプロイを行うため、手動でデプロイする必要がありません。

  さらに、アプリケーションはローカルで動作するため、途中で止まってしまうなどの可能性が低いです。

- **スケーリングの柔軟性が高い**
  KubernetesではPodをスケーリングすることが可能です。

  そのため、使用する用途に応じてPodの自動生成などができ、柔軟にリソースを増減できます。

- **セキュリティを強化できる障害に強い**
  Kubernetesは周辺技術を理解できることでセキュリティの強化が可能です。

  例えばKubernetesはAPIを用いて操作するため、APIを使用するユーザーの認証・権限の制限をすることでセキュリティ性能を高めることができます。

  また、Podやコンテナなどのオブジェクトに対するリソース制限を加えることでもセキュリティの強化が期待できます。

  このように様々なアクセス制限や機能制限、権限の設定を細かくできるため、使用用途に応じてセキュリティレベルを設定できます。

- **障害に強い**
  Kubernetesは自動回復機能を備えています。

  コンテナがダウンした場合や誤って削除してしまった場合にも、コンテナを自動で回復することができます。

  この自動回復機能は、各種の障害に対する強さとなります。

  利用者は自動回復機能を頼りに、落ち着いて操作することができます。

#### デメリット

- **物理サーバーの台数が増える傾向にあり初期投資がかかりやすい**
  Kubernetesでは、実行マシンとしての「ワーカーノード」と管理マシンとしての「マスターノード」が必要です。

  実際の運用ではワーカーノードとマスターノードを別の物理マシンとして用意する必要があり、ノードの数に比例して必要となる物理サーバーの数も増加します。

  したがって、構成や規模によってはオンプレミス環境のようにある程度の初期投資が必要になる傾向があります。

- **更新頻度が高く、学習コストも高い**
  Kubernetesは機能が豊富で、進化の過程にあるツールでもあります。  
  したがって、Kubernetesを上手く活用するためには継続的なキャッチアップが必要になります。

- **使いこなすには目的を明確にする必要がある**

  Kubernetesは豊富な機能を持っています。

  各機能を活用することで柔軟な設定が実現でき、拡張性も高いです。その分、利用機能の選択や設定にも多くの選択肢があります。

  Kubernetesを用いて何を実現したいのか、運用管理の形を事前に明確にして利用する機能や設定内容を定める必要があります。

## 💻️ ハンズオン

### 起動

以下コマンドを実施し、minicube を起動します。

```bash
minikube start

W0525 17:12:44.569352   62706 main.go:291] Unable to resolve the current Docker CLI context "default": context "default": context not found: open /Users/wan0ri/.docker/contexts/meta/37a8eec1ce19687d132fe29051dca629d164e2c4958ba141d5f4133a33f0688f/meta.json: no such file or directory
W0525 17:12:44.569404   62706 main.go:292] Try running `docker context use default` to resolve the above error
😄  Darwin 15.5 (arm64) 上の minikube v1.36.0
✨  docker ドライバーが自動的に選択されました
📌  root 権限を持つ Docker Desktop ドライバーを使用
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.47 ...
💾  ロード済み Kubernetes v1.33.1 をダウンロードしています...
    > preloaded-images-k8s-v18-v1...:  327.15 MiB / 327.15 MiB  100.00% 8.95 Mi
    > gcr.io/k8s-minikube/kicbase...:  463.69 MiB / 463.69 MiB  100.00% 1.79 Mi
🔥  Creating docker container (CPUs=2, Memory=7789MB) ...
🐳  Docker 28.1.1 で Kubernetes v1.33.1 を準備しています...
    ▪ 証明書と鍵を作成しています...
    ▪ コントロールプレーンを起動しています...
    ▪ RBAC のルールを設定中です...
🔗  bridge CNI (コンテナーネットワークインターフェース) を設定中です...
🔎  Kubernetes コンポーネントを検証しています...
    ▪ gcr.io/k8s-minikube/storage-provisioner:v5 イメージを使用しています
🌟  有効なアドオン: default-storageclass, storage-provisioner
🏄  終了しました！kubectl がデフォルトで「minikube」クラスターと「default」ネームスペースを使用するよう設定されました
```

Docker Desktop ＞ Container を確認すると、 `minikube` が起動できていることが確認できました。

![Docker Desktop](/images/kubernetes-tutorial/DockerDesktop.png)

ターミナルにて、自端末のIPアドレス(IPv4)を事前に調べておきます。

```bash
ifconfig
```

(参考)  
<https://qiita.com/nlog2n2/items/1d1358f6913249f3e186>

Remote SSHのconfig設定は、以下で進めていきます。

```config
Host minikube
    HostName 127.0.0.1
    User root
```

開発コンテナー ＞ その他のコンテナー に、起動している `minikube` があるので、  
起動している `minikube` へ接続します。

![Remote SSH](/images/kubernetes-tutorial/editor_check.png)

接続が完了したら下記の表示になるので、 `root` を選択します。

![root](/images/kubernetes-tutorial/minikube-root.png)

下記のようになりましたら、接続と準備の方は完了になります。

![setup](/images/kubernetes-tutorial/minikube-access-after.png)

### Hello World！(Kubernetes)

お試しに、テキストエディタ内でターミナルを起動してKubernetes上に `hello-world` Dockerコンテナを立ち上げるところまで実施します。

コンテナを起動する前に注意点があります。  
そのままコンテナ起動コマンドを実行しても、コンテナ内に `kubectl` コマンドが登録されていないため立ち上げることができません。  
そこで、下記コマンドを実行して起動できるように準備していきます。

#### 準備(minikubeコンテナ内にkubectlをインストール)

コンテナ内でkubectlを使いたい場合は、以下の手順でインストールできます：

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

```bash
chmod +x kubectl
```

```bash
mv kubectl /usr/local/bin/
```

#### kubeconfigの設定

kubectlをインストール後、kubeconfigを設定する必要があります：

```bash
mkdir -p ~/.kube
```

```bash
cp /etc/kubernetes/admin.conf ~/.kube/config
```

```bash
chown $(id -u):$(id -g) ~/.kube/config
```

#### 確認

```bash
kubectl get nodes

NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   85m   v1.33.1
```

#### コンテナ起動

```bash
kubectl run hello-world --image hello-world --restart=Never

pod/hello-world created
```

上記コマンドをブロックごとに切り出して解説します：

- kubectl
  Kubernetesのコマンド

- run
  起動コマンド

- hello-world
  KubernetesのPod名

- --image
  Kubernetesのイメージ名の指定

- hello-world
  Kubernetesのイメージ名

- --restart=Never
  起動方法の指定※

※起動方法には `Always` や `OnFailure`がある。以下参考。  
<https://qiita.com/ssc-wkani/items/ee0930001c0663358392>

#### 起動後の確認

出来上がっていることを確認するため、`kubectl get pod` コマンドを実行します。

```bash
kubectl get pod

NAME          READY   STATUS      RESTARTS   AGE
hello-world   0/1     Completed   0          10m
```

#### ログ確認

ログを確認するため、 `kubectl logs pod/hello-world` コマンドを実行します。

```bash
kubectl logs pod/hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

#### 作成したコンテナの削除

作成したコンテナの削除を実施します。

```bash
kubectl delete pod/hello-world

pod "hello-world" deleted
```

### Kubernetesリソース

Kubernetesの主なリソースは以下の通りとなります。

| 分類         | 種別                                        |
| ------------ | ------------------------------------------- |
| ワークロード | Pod / ReplicaSet / Deployment / StatefulSet |
| サービス     | Service / Ingress                           |
| 設定         | ConfigMap / Secret                          |
| ストレージ   | PersistentVolume / PersistentVolumeClaim    |

#### ワークロード

- Pod
  最小単位。Dockerコンテナの集合。

- ReplicaSet
  Podの集合。Podをスケールできる。

- Deployment
  ReplicaSetの集合。ReplicaSetの世代管理ができる。

- StatefulSet
  Podの集合。Podをスケールする際の名前が一定。

#### サービス

- Service
  外部公開。名前解決。L4ロードバランサー※1。
- Ingress
  外部公開。L7ロードバランサー※2。

※1  
::: details L4ロードバランサー（レイヤー4）
**動作レベル**: トランスポート層（TCP/UDP）  
**特徴**：

- **IPアドレスとポート番号**のみを見て負荷分散を行う
- パケットの中身（HTTPヘッダーやコンテンツ）は見ない
- 高速で処理が軽い
- プロトコルに依存しない
  **負荷分散の例**

```bash
クライアント → L4ロードバランサー → サーバー群
   (IP:Port で振り分け)
```

**使用例**
・データベースの負荷分散  
 ・ゲームサーバーの負荷分散  
 ・単純なWebサーバーの負荷分散  
:::

※2  
::: details L7ロードバランサー（レイヤー7）
**動作レベル**: アプリケーション層（HTTP/HTTPS）  
**特徴**：

- **HTTPヘッダー、URL、コンテンツ**を解析して負荷分散
- より高度なルーティングが可能
- 処理が重い（パケットの中身を解析するため）
- アプリケーション固有の機能を提供
  **負荷分散の例**

```bash
クライアント → L7ロードバランサー → サーバー群
            (URL、ヘッダー等で振り分け)
```

:::

#### 設定

- ConfigMap
  設定情報を保存するためのリソース。

- Secret
  機微情報を保存するためのリソース。

#### ストレージ

- PersistentVolume
  永続データの実態。 ストレージへの接続情報、ストレージの抽象化を行う。
- PersistentVolumeClaim
  永続データの要求。 抽象化されたストレージの要求を行う。

### Kubernetesネットワーク

#### NodeとPod

リソースは各ワーカーノードに分散配置されます。

上記について図を書こうと思いましたが、私には図を書く環境が整っていないためAIにアスキーアートで出してもらいました。

リソース分散の特徴として：  
　• 各WorkerノードにPodが分散配置される  
　• Schedulerがリソース使用量を考慮して最適なノードを選択  
　• 障害時の影響を最小化（単一障害点の回避）  
　• 負荷分散によるパフォーマンス向上

が、下図からある程度理解することができるかと思います。

```text
Kubernetes Cluster - リソースの分散配置
================================================

Control Plane (Master Node)
┌─────────────────────────────┐
│  • API Server               │
│  • etcd                     │
│  • Scheduler                │
│  • Controller Manager       │
└─────────────────────────────┘
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼

Worker Node 1    Worker Node 2    Worker Node 3
┌───────────┐    ┌───────────┐    ┌───────────┐
│ kubelet   │    │ kubelet   │    │ kubelet   │
│ kube-proxy│    │ kube-proxy│    │ kube-proxy│
│           │    │           │    │           │
│ ┌───────┐ │    │ ┌───────┐ │    │ ┌───────┐ │
│ │ Pod A │ │    │ │ Pod C │ │    │ │ Pod E │ │
│ │ App1  │ │    │ │ App2  │ │    │ │ App3  │ │
│ └───────┘ │    │ └───────┘ │    │ └───────┘ │
│           │    │           │    │           │
│ ┌───────┐ │    │ ┌───────┐ │    │ ┌───────┐ │
│ │ Pod B │ │    │ │ Pod D │ │    │ │ Pod F │ │
│ │ DB1   │ │    │ │ Cache │ │    │ │ Web   │ │
│ └───────┘ │    │ └───────┘ │    │ └───────┘ │
└───────────┘    └───────────┘    └───────────┘
```

#### 2つの異なるネットワーク

Kubernetesには、2つの異なるネットワークが存在しています。

- クラスタネットワーク
  クラスタネットワークへ外へ直接アクセスすることができない。
- 外部ネットワーク
  管理端末は外部ネットワークに接続。

これも例に及んでAIにアスキーアートで出してもらいました。

**▼主要な特徴と説明**  
　●クラスタネットワーク

- **Pod Network**: Pod間の直接通信用（CNIプラグインが管理）
- **Service Network**: 仮想IPによる負荷分散とサービス発見
- **Node Network**: 物理的なノード間通信

**▼外部ネットワーク**
(例)

- **LoadBalancer**: クラウドプロバイダーの外部ロードバランサー
- **NodePort**: 各ノードの特定ポートで外部公開
- **Ingress**: HTTP/HTTPSレベルでのルーティング制御

**▼ネットワーク分離**
(例)

- **Namespace**: 論理的な分離
- **NetworkPolicy**: トラフィック制御とセキュリティ
- **CNI Plugin**: Pod間通信の実装（Flannel, Calico, Weave等）

```text
通信フロー例: 外部ユーザー → Pod への通信
═══════════════════════════════════════════════

1. 外部からの通信
   Internet User (203.0.113.100)
   │
   │ HTTP Request
   ▼
   Load Balancer (203.0.113.10:80)

2. クラスタ内への転送
   │
   │ Forward to NodePort/Ingress
   ▼
   Ingress Controller (10.96.1.100:80)
   │
   │ Route based on rules
   ▼
   Service A (10.96.1.1:80)

3. Pod への負荷分散
   │
   │ Load Balance (kube-proxy)
   ├─────────────┬─────────────┐
   ▼             ▼             ▼
   Pod A         Pod C         Pod E
   (10.244.1.10) (10.244.2.10) (10.244.3.10)
   Node 1        Node 2        Node 3

4. Pod間通信 (同一Service内)
   Pod A (10.244.1.10) ←→ Pod C (10.244.2.10)
   │                        │
   │ CNI Plugin (Flannel/Calico/Weave)
   │                        │
   Node 1 ←─────────────────→ Node 2
   (192.168.1.10)           (192.168.1.11)

```

コンテナのアクセス方法として、以下の2つのやり方があります：

- コンテナに直接ログイン
- 踏み台サーバ経由でコンテナへアクセス
- サービス経由でアクセスする

## 基本操作

### リソースの作成・変更・確認・削除

#### 作成手順

「**定義作成**」　→　「**定義適用**」  
の順で作成していきます。

```text
マニフェストファイル作成：YAMLファイルを作成
↓
Kubernetes反映：kubectlコマンドを利用して反映
```

**▼マニフェストファイルの作成**  
YAMLファイルにリソースの定義を記載します。

(例)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
    env: test
spec:
  containers:
    - name: nginx-container
      image: nginx;1.17.2-alpine
```

**▼リソースの作成・変更**
マニフェストファイルを指定してリソースを作成、変更します。

```sh
kubectl apply -f <filename>
```

◯オプション  
 `-f <filename>` ： マニフェストファイルパス

**▼リソース確認コマンド**
指定したリソースの状態を確認します。

```sh
lubectl get [-f <filename>] [type]
```

◯オプション  
`-f <filename>` ： マニフェストファイルパス  
`TYPE` ： リソース種別（pod, resplicaset など）

**▼リソース削除コマンド**
指定したリソースを削除します。

```sh
kubectl delete [-f <filename>] [TYPE / NAME] [-o [wide|yaml]]
```

◯オプション  
`-f <filename>` ： マニフェストファイルパス
`TYPE / NAME -o [wide|yaml]` ： 出力形式を指定します。  
 ※wide：追加情報の表示, yaml：YAML形式で表示

#### 💡 Tips

学習途中で `minikube` を途中で停止し後日再開する際、テキストエディタの操作方法を以下の手順で進めていくと学習を再開できます。

**※Remote SSHの拡張機能をインストールしている、かつ、ターミナルでminikubeを起動している前提** 1.画面左下のRemote SSHを押下  
2.実行中のコンテナーにアタッチを押下

![nimikube-open-tips](/images/kubernetes-tutorial/minikube-open-tips.png)

#### 演習

作成手順で記載したコマンドを、実際に演習形式で進めていこうと思います。

📖 お品書き  
1.`hello-world` コンテナを含むPodを作成
2.Podが起動していることを確認
3.Podを削除

**1.`hello-world` コンテナを含むPodを作成**  
・まずは `root` 直下にいることを確認します。

```sh
root@minikube:~# pwd
/root
root@minikube:~#
```

・作業用ディレクトリを作成します。名前は何でもよいですが、とりあえず `tutorial` ディレクトリとしておきます。

```sh
root@minikube:~# mkdir ./tutorial
```

　・`root` 直下に `tutorial` ディレクトリが作成されていることを確認します。

```sh
root@minikube:~# ls -l
total 4
drwxr-xr-x 2 root root 4096 May 27 23:05 tutorial
root@minikube:~#
```

　・`tutorial` 直下に `pod.yml` を作成します。(注釈あり)

```yml
# Kubernetes APIのバージョンを指定（Pod リソースはv1）
apiVersion: v1
# リソースの種類を指定（この場合はPod）
kind: Pod
# Podのメタデータ情報
metadata:
  # Podの名前
  name: nginx
  # Podが作成される名前空間（省略可能、デフォルトはdefault）
  namespace: default
  # Podに付与するラベル（セレクタやフィルタリングに使用）
  labels:
    app: nginx # アプリケーション名を示すラベル
    env: test # 環境を示すラベル（test, prod, devなど）
# Podの仕様・設定
spec:
  # Pod内で実行するコンテナのリスト
  containers:
    # コンテナの名前
    - name: nginx-container
      # 使用するDockerイメージとタグ
      image: nginx:1.17.2-alpine
```

**2.Podが起動していることを確認**
・`tutorial` ディレクトリに移動します。

```sh
root@minikube:~# cd tutorial
root@minikube:~/tutorial#
```

・リソースを作成します。

```sh
root@minikube:~/tutorial# kubectl apply -f pod.yml
pod/nginx created
root@minikube:~/tutorial#
```

・作成されたリソースの確認を実施します。

```sh
root@minikube:~/tutorial# kubectl get pod
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          52s
root@minikube:~/tutorial#
```

**3.Podを削除**
・リソースを削除します。

```sh
root@minikube:~/tutorial# kubectl delete -f pod.yml
pod "nginx" deleted
root@minikube:~/tutorial#
```

・リソースの削除確認を実施します。

```sh
root@minikube:~/tutorial# kubectl get pod
No resources found in default namespace.
root@minikube:~/tutorial#
```
