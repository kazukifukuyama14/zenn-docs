---
title: "ミリしらKubernetes〜ド初心者がKubernetesをある程度理解するまでの記録〜"
emoji: "🕸️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Kubernetes, Docker]
published: true
published_at: 2027-05-31 09:00
---

## 🎯 はじめに

筆者はDockerを業務で触ったことはございますが、Kubernetesを業務で触ったことがございません。  
そこで、  
**Kubernetesを1ミリもしらない状態でアウトプットしていけばどこまで理解することができるのか**  
をハンズオン形式で実践して理解を深めていきたいと思います。

## 🛠️ 環境構築

### Docker

Dockerはコンテナを作成・管理するために必要です。  
以下のリンクから自分の環境に合わせてインストールします。
<https://docs.docker.com/engine/install/>

---

### IDE（開発環境）

Visual Studio Codeなど慣れているものでいいとは思いますが、私はCursorを使用していきます。  
<https://www.cursor.com/ja>

また、テキストエディタには拡張機能「**Remote SSH**」が必要ですのでそちらをインストールしておきます。  
<https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh>

---

### Kubernetes CLI（kubectl）

下記コマンドを入力し、インストールしていきます。

```bash
brew install kubernetes-cli
```

```bash
brew install minikube
```

(参考)  
<https://qiita.com/Asaminnn/items/6d414d4776d964e94e96>

## 📚 Kubernetesの基礎知識

### Kubernetesとは

「**Kubernetes**」とは、  
**コンテナ型仮想化技術を対象とした運用管理、自動化のツール、コンテナオーケストレーションツールの一つ**  
のことです。

---

### DockerとKubernetesの違い

コンテナと聞くとDockerを思い浮かべますが、DockerとKubernetesでは違いがあります。

**Docker**は簡単にコンテナの作成や削除ができるソフトウェアツールです。  
必要なパッケージのコード化や環境の再配布、チーム開発時の環境の統一などに使用されます。

一方、**Kubernetes**は複数のコンテナを用いた開発に使用するソフトウェアツールです。  
各コンテナの状態を確認し、問題のあるコンテナを再起動する、といった管理が可能です。

#### 管理単位の違い

この二つのツールの間では管理単位にも違いがあります。

**Docker**はある**マシン上の一つ一つのコンテナが最小の単位**で、コンテナの乗ったマシンであるノードも管理の単位です。

一方の**Kubernetes**では最小の単位は**Pod**です。  
Podは一つ以上のコンテナを持ちますが、1Podに1コンテナである場合が多いです。  
**1つ以上のPodを持つマシンがノード（ワーカーノード）** です。

より大きい単位としてはクラスタがあります。  
**クラスタはノードの集合体**で、1つ以上のワーカーノードとそれを管理するマスターノードを持ちます。

|          | Kubernetes            | Docker                       |
| -------- | --------------------- | ---------------------------- |
| **用途** | コンテナ管理          | アプリケーションのコンテナ化 |
| **単位** | Pod、ノード、クラスタ | コンテナ、ノード             |

---

### Kubernetesのメリット・デメリット

Kubernetesは、非常に効率よく大規模なITインフラを運用・管理できるツールです。  
しかし、万能のツールというわけではなく、実際には以下のようなメリット・デメリットを持ちます。

#### ✅ メリット

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

- **セキュリティを強化できる**  
  Kubernetesは周辺技術を理解できることでセキュリティの強化が可能です。

  例えばKubernetesはAPIを用いて操作するため、APIを使用するユーザーの認証・権限の制限をすることでセキュリティ性能を高めることができます。

  また、Podやコンテナなどのオブジェクトに対するリソース制限を加えることでもセキュリティの強化が期待できます。

  このように様々なアクセス制限や機能制限、権限の設定を細かくできるため、使用用途に応じてセキュリティレベルを設定できます。

- **障害に強い**  
  Kubernetesは自動回復機能を備えています。

  コンテナがダウンした場合や誤って削除してしまった場合にも、コンテナを自動で回復することができます。

  この自動回復機能は、各種の障害に対する強さとなります。

  利用者は自動回復機能を頼りに、落ち着いて操作することができます。

#### ❌ デメリット

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

## 💻 ハンズオン実践

### minikubeの起動

以下コマンドを実施し、minikubeを起動します。

```bash
minikube start
```

実行結果：

```bash
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

### Remote SSH接続の設定

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

---

### Hello World！（Kubernetes初回実行）

お試しに、テキストエディタ内でターミナルを起動してKubernetes上に `hello-world` Dockerコンテナを立ち上げるところまで実施します。

コンテナを起動する前に注意点があります。  
そのままコンテナ起動コマンドを実行しても、コンテナ内に `kubectl` コマンドが登録されていないため立ち上げることができません。  
そこで、下記コマンドを実行して起動できるように準備していきます。

#### 事前準備：minikubeコンテナ内にkubectlをインストール

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

#### 動作確認

```bash
kubectl get nodes
```

実行結果：

```bash
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   85m   v1.33.1
```

#### hello-worldコンテナの起動

```bash
kubectl run hello-world --image hello-world --restart=Never
```

実行結果：

```bash
pod/hello-world created
```

**コマンド解説：**

- `kubectl` : Kubernetesのコマンド
- `run` : 起動コマンド
- `hello-world` : KubernetesのPod名
- `--image` : Kubernetesのイメージ名の指定
- `hello-world` : Kubernetesのイメージ名
- `--restart=Never` : 起動方法の指定※

※起動方法には `Always` や `OnFailure`がある。以下参考。  
<https://qiita.com/ssc-wkani/items/ee0930001c0663358392>

#### 起動状況の確認

出来上がっていることを確認するため、`kubectl get pod` コマンドを実行します。

```bash
kubectl get pod
```

実行結果：

```bash
NAME          READY   STATUS      RESTARTS   AGE
hello-world   0/1     Completed   0          10m
```

#### ログの確認

ログを確認するため、 `kubectl logs pod/hello-world` コマンドを実行します。

```bash
kubectl logs pod/hello-world
```

実行結果：

```bash
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

#### リソースのクリーンアップ

作成したコンテナの削除を実施します。

```bash
kubectl delete pod/hello-world
```

実行結果：

```bash
pod "hello-world" deleted
```

## 📋 Kubernetesリソースの理解

### リソースの分類

Kubernetesの主なリソースは以下の通りとなります。

| 分類         | 種別                                        |
| ------------ | ------------------------------------------- |
| ワークロード | Pod / ReplicaSet / Deployment / StatefulSet |
| サービス     | Service / Ingress                           |
| 設定         | ConfigMap / Secret                          |
| ストレージ   | PersistentVolume / PersistentVolumeClaim    |

---

### ワークロード系リソース

- **Pod**  
  最小単位。Dockerコンテナの集合。

- **ReplicaSet**  
  Podの集合。Podをスケールできる。

- **Deployment**  
  ReplicaSetの集合。ReplicaSetの世代管理ができる。

- **StatefulSet**  
  Podの集合。Podをスケールする際の名前が一定。

---

### サービス系リソース

- **Service**  
  外部公開。名前解決。L4ロードバランサー※1。
- **Ingress**  
  外部公開。L7ロードバランサー※2。

:::details L4ロードバランサー（レイヤー4）
**動作レベル**: トランスポート層（TCP/UDP）  
**特徴**：

- **IPアドレスとポート番号**のみを見て負荷分散を行う
- パケットの中身（HTTPヘッダーやコンテンツ）は見ない
- 高速で処理が軽い
- プロトコルに依存しない

**負荷分散の例**:

```bash
クライアント → L4ロードバランサー → サーバー群
   (IP:Port で振り分け)
```

**使用例**  
・データベースの負荷分散  
・ゲームサーバーの負荷分散  
・単純なWebサーバーの負荷分散  
:::

:::details L7ロードバランサー（レイヤー7）
**動作レベル**: アプリケーション層（HTTP/HTTPS）  
**特徴**：

- **HTTPヘッダー、URL、コンテンツ**を解析して負荷分散
- より高度なルーティングが可能
- 処理が重い（パケットの中身を解析するため）
- アプリケーション固有の機能を提供

**負荷分散の例**:

```bash
クライアント → L7ロードバランサー → サーバー群
            (URL、ヘッダー等で振り分け)
```

:::

---

### 設定系リソース

- **ConfigMap**  
  設定情報を保存するためのリソース。

- **Secret**  
  機微情報を保存するためのリソース。

---

### ストレージ系リソース

- **PersistentVolume**  
  永続データの実態。ストレージへの接続情報、ストレージの抽象化を行う。
- **PersistentVolumeClaim**  
  永続データの要求。抽象化されたストレージの要求を行う。

## 🌐 Kubernetesネットワークの理解

### ノードとPodの分散配置

リソースは各ワーカーノードに分散配置されます。

**リソース分散の特徴**  
• 各WorkerノードにPodが分散配置される  
• Schedulerがリソース使用量を考慮して最適なノードを選択  
• 障害時の影響を最小化（単一障害点の回避）  
• 負荷分散によるパフォーマンス向上

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

---

### ネットワークの種類

Kubernetesには、2つの異なるネットワークが存在しています。

- **クラスタネットワーク**  
  クラスタ内部のネットワーク。外部から直接アクセスすることができない。
- **外部ネットワーク**  
  管理端末は外部ネットワークに接続。

#### クラスタネットワークの構成要素

- **Pod Network**: Pod間の直接通信用（CNIプラグインが管理）
- **Service Network:** 仮想IPによる負荷分散とサービス発見
- **Node Network:** 物理的なノード間通信

#### 外部ネットワークとの接続方法

- **LoadBalancer**: クラウドプロバイダーの外部ロードバランサー
- **NodePort**: 各ノードの特定ポートで外部公開
- **Ingress**: HTTP/HTTPSレベルでのルーティング制御

#### ネットワーク分離とセキュリティ

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

---

### コンテナへのアクセス方法

コンテナのアクセス方法として、以下の方法があります：

- コンテナに直接ログイン
- 踏み台サーバ経由でコンテナへアクセス
- サービス経由でアクセスする

## ⚙️ Kubernetesの基本操作

### リソースの作成・変更・確認・削除

#### 基本的な作成手順

「**定義作成**」　→　「**定義適用**」  
の順で作成していきます。

```text
マニフェストファイル作成：YAMLファイルを作成
↓
Kubernetes反映：kubectlコマンドを利用して反映
```

#### マニフェストファイルの作成

YAMLファイルにリソースの定義を記載します。

**例：**

```yaml:pod.yml
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
      image: nginx:1.17.2-alpine
```

**リソースの作成・変更**:
マニフェストファイルを指定してリソースを作成、変更します。

```bash
kubectl apply -f <filename>
```

**オプション：**  
`-f <filename>` ： マニフェストファイルパス

**リソース確認コマンド:**
指定したリソースの状態を確認します。

```bash
kubectl get [-f <filename>] [TYPE]
```

**オプション：**  
`-f <filename>` ： マニフェストファイルパス  
`TYPE` ： リソース種別（pod, replicaset など）

**リソース削除コマンド**
指定したリソースを削除します。

```bash
kubectl delete [-f <filename>] [TYPE/NAME] [-o [wide|yaml]]
```

**オプション：**  
`-f <filename>` ： マニフェストファイルパス  
`TYPE/NAME -o [wide|yaml]` ： 出力形式を指定します。  
※wide：追加情報の表示, yaml：YAML形式で表示

---

### 💡 学習継続のためのTips

学習途中で `minikube` を途中で停止し後日再開する際、テキストエディタの操作方法を以下の手順で進めていくと学習を再開できます。

**※前提条件：Remote SSHの拡張機能をインストールしている、かつ、ターミナルでminikubeを起動している**:

1. 画面左下のRemote SSHを押下
2. 実行中のコンテナーにアタッチを押下

![minikube-open-tips](/images/kubernetes-tutorial/minikube-open-tips.png)

---

### 実践演習：Podの作成から削除まで

作成手順で記載したコマンドを、実際に演習形式で進めていこうと思います。

**📖 演習内容**:

1. nginxコンテナを含むPodを作成
2. Podが起動していることを確認
3. Podを削除

#### Step 1: Podの作成

**現在のディレクトリを確認**:

```bash
root@minikube:~# pwd
/root
root@minikube:~#
```

**作業用ディレクトリを作成**:

名前は何でもよいですが、とりあえず `tutorial` ディレクトリとしておきます。

```bash
root@minikube:~# mkdir ./tutorial
```

**ディレクトリの作成確認**:

`root` 直下に `tutorial` ディレクトリが作成されていることを確認します。

```bash
root@minikube:~# ls -l
total 4
drwxr-xr-x 2 root root 4096 May 27 23:05 tutorial
root@minikube:~#
```

**マニフェストファイルの作成**:

`tutorial` 直下に `pod.yml` を作成します。

:::details pod.yml

```yaml:tutorial/pod.yml
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

:::

#### Step 2: Podの起動確認

**作業ディレクトリに移動**:

```bash
root@minikube:~# cd tutorial
root@minikube:~/tutorial#
```

**リソースの作成**:

```bash
root@minikube:~/tutorial# kubectl apply -f pod.yml
pod/nginx created
root@minikube:~/tutorial#
```

**作成されたリソースの確認**:

```bash
root@minikube:~/tutorial# kubectl get pod
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          52s
root@minikube:~/tutorial#
```

#### Step 3: Podの削除

**リソースの削除**:

```bash
root@minikube:~/tutorial# kubectl delete -f pod.yml
pod "nginx" deleted
root@minikube:~/tutorial#
```

**削除の確認**:

```bash
root@minikube:~/tutorial# kubectl get pod
No resources found in default namespace.
root@minikube:~/tutorial#
```

---

### マニフェストファイルの種類

前段では、Podの作成・確認・削除を行いました。  
ここでは、マニフェストファイルの種類について記載します。

#### そもそもマニフェストファイルって何？

- インデントで構造を定義するYAMLファイル形式のこと。
- 種別、メタデータ、コンテナの3構成で定義されている。
  ※リソースによって定義の中身に変動あり

前段で説明しました `pod.yml` について、改めて詳細の説明を記載したいと思います。

(例)：

::: details tutorial/pod.yml

```yaml:tutorial/pod.yml
# kind と apiVersion
apiVersion: v1
kind: Pod
# metadata と name、namespace、labels
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx # アプリケーション名を示すラベル
    env: test # 環境を示すラベル（test, prod, devなど）
# コンテナの設定
spec:
  containers:
    - name: nginx-container
      image: nginx:1.17.2-alpine
```

:::

#### kind と apiVersion

- リソース種別。
- kindによって、apiVersionの値が変わる。

#### metadata と name、namespace、labels

- Pod名は名前空間と合わせて一意になるようにする。
  ※namespaceを省略した場合は、defaultが適用される。
- labelsはセレクタやフィルタリングに使用される。
- labelsは複数指定可能。

#### spec

- コンテナの設定。
- 複数のコンテナを指定可能。
- コンテナの設定は複数指定可能。

---

例題には記載しておりませんが、コンテナの設定には以下のようなものがあります。

::: details pod.yml

```yaml:pod.yml
〜説明用のため、specから上は省略〜
spec:
  containers:
    - name: nginx
      image: nginx:1.17.2-alpine
      # command と args の設定
      command: ["/bin/sh"]
      args: ["-c", "while true; do echo hello; sleep ${DELAY}; done"]
      # env の設定
      env:
      - name: "DELAY"
        value: "5"
```

:::

#### command と args

- コンテナの起動時に実行するコマンドと引数を指定する。
- commandはコマンド名、argsは引数。
- commandは1つ、argsは複数指定可能。
- commandとargsはどちらか一方のみ指定可能。
- commandとargsの両方を指定した場合、commandのコマンドが実行される。
- 例題での設定では、コンテナが起動して`から5秒ごとに「hello」と表示するような例題。

#### env

- コンテナ内で使用する環境変数を設定する。
- nameは環境変数名、valueは値。
- envは複数指定可能。
- envはcommandとargsの後に記載する。
- 例題での設定では、コンテナ内で環境変数「DELAY」を5に設定している。

---

### kindに応じた apiVersion の確認

再掲になりますが、マニフェストファイルには **kind に応じた apiVersion を指定** する必要があります。  
そこで、kindに応じた apiVersion を確認する方法を記載します。

結論になりますが、 Kubernetes公式ドキュメントの[Kubernetes APIリファレンス](https://kubernetes.io/docs/reference/kubernetes-api/)を参照すると確認できます。

例えば前段でも記載した `pod.yml` の apiVersion は `v1` となっており、リファレンスでは下記に該当します。  
[kubeconfig(v1)](https://kubernetes.io/docs/reference/config-api/kubeconfig.v1/)

---

### Pod に入ってコマンド実行

起動中のコンテナへ入り、コンテナから出るためのコマンドを実行していきます。  
各種コマンドは以下の通りです。

**▼コンテナへ入る**:  
この構文は以前のバージョンの構文のため誤りとなります。  
最新バージョンの構文は後述で記載しますので、下記は無視してください🙇‍♂️

```bash
kubectl exec -it POD sh
```

・引数  
POD：中に入りたいPod名

**▼コンテナから出る**:

・プロセスを終了してからログアウト

```bash
exit
```

・プロセスを残したままコンテナからログアウト

```bash
[ctrl + P] + [ctrl + Q]
```

#### 演習

**📖 演習内容**:

```txt
1. `CentOS` と `nginx` のコンテナを含むマニフェストファイルを作成
2. `CentOS` と ``nginx` のPodを起動
3. PodのIPアドレスを確認
4. 起動した `CentOS` コンテナと `nginx` コンテナ内に入る
5. コンテナから出る
6. `CentOS` と ``nginx` のPodを削除
```

1.まずは任意のテキストエディタからSSHで接続し、マニフェストファイルを作成します。  
※前段の演習で使用した `tutorial` ディレクトリを使いまわし、その配下に `pods.yml` を作成します。

::: details tutorial/pods.yml

```yaml:tutorial/pods.yml
# CentOS Pod の定義
apiVersion: v1
kind: Pod
metadata:
  name: debug
  namespace: default
  labels:
    env: test
spec:
  containers:
    - name: debug
      image: centos:7
      command:
        - sh    # シェルを起動
        - -c    # コマンド文字列を実行するオプション
      # commandに渡す引数
      args:
        - |
          while true
          do
      # 環境変数DELAYの値だけスリープする
            sleep ${DELAY}
          done
      # コンテナ内で使用する環境変数
      env:
        - name: "DELAY"
          value: "5"

---  # YAML文書の区切り文字

# nginx Pod の定義
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
    env: test
spec:
  containers:
    - name: nginx-container
      image: nginx:1.17.2-alpine
```

:::

**※注意**:  
マニフェストファイルに複数の定義を記載する際、必ず **`---`** で区切る必要があります。  
そうしなければ、リソースの作成が失敗しますのでご注意ください。

2.作成したマニフェストファイルを起動させます。

テキストエディタのコマンドラインから、下記コマンドを実行して `tutorial` ディレクトリに移動します:

```bash
root@minikube:~# cd tutorial/
root@minikube:~/tutorial#
```

リソースの作成を実行します:

```bash
root@minikube:~/tutorial# kubectl apply -f pods.yml
pod/debug created
pod/nginx created
root@minikube:~/tutorial#
```

3.作成したPodのIPアドレスを確認します。  
`-o wide` オプションで、PodのIPアドレスを確認できます:

```bash
root@minikube:~/tutorial# kubectl get pod -o wide
NAME    READY   STATUS             RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
debug   0/1     ImagePullBackOff   0          3m57s   10.244.0.18   minikube   <none>           <none>
nginx   1/1     Running            0          3m57s   10.244.0.17   minikube   <none>           <none>
root@minikube:~/tutorial#
```

4.次に、本題のコンテナへの入り方を記載します。  
`debug` コンテナに入ります。

```bash
root@minikube:~/tutorial# kubectl exec -it debug sh
error: exec [POD] [COMMAND] is not supported anymore. Use exec [POD] -- [COMMAND] instead
See 'kubectl exec -h' for help and examples
root@minikube:~/tutorial#
```

・・・？
エラーが出てしまいました。  
`kubectl exec` コマンドが使えないようです。  
これは、Kubernetes v1.24から 新しいバージョンのkubectlでは、コマンドの前に `--` を付ける必要があるのだそうです。  
`kubectl exec -it POD sh` コマンドの代わりに、 `kubectl exec -it POD -- ssh` を使用します。  
これで、コンテナ内に入ることがでるはずです。

```bash
root@minikube:~/tutorial# kubectl exec -it debug -- sh
sh-4.2#
```

入れました！  
`exit` コマンドでコンテナから出て、残りの `nginx` コンテナへのアクセスを確認します。

```bash
root@minikube:~/tutorial# kubectl exec -it nginx -- sh
/ #
```

**補足：コンテナへ入る(最新バージョン)**:

```bash
kubectl exec -it POD -- sh
```

・引数  
POD：中に入りたいPod名

`--` は、**コマンドの引数をPod内のシェルに渡すためのオプションです**。
`kubectl exec` コマンドの構文変更について、詳細は[こちら](https://kubernetes.io/ja/docs/tasks/debug/debug-application/get-shell-running-container/)をご確認ください。

```txt
この変更は、Kubernetesのセキュリティと一貫性を向上させるために導入されました：

- `--` はコマンドとオプションを明確に分離します
- コマンドインジェクション攻撃のリスクを軽減します
- 他のkubectlコマンドとの一貫性を保ちます
```

5.コンテナから `exit` コマンドでコンテナから出ます。

6.最後に、作成したPodを削除します。

```bash
root@minikube:~/tutorial# kubectl delete -f pods.yml
pod "debug" deleted
pod "nginx" deleted
root@minikube:~/tutorial#
```

---

### Pod とホスト間でのファイルのやり取り

Pod内のファイルをホスト側にコピーするには、`kubectl cp` コマンドを使用します。  
下記はそれぞれのパターンでのコマンドの例です。  
共通して、PODNAMEの後には転送元と転送先のファイルパスを記載する前に、**`:`** を付けてください。

**▼ホスト側のファイルをPod内にコピー**
基本構文:

```bash
kubectl cp SRC PODNAME:DEST
```

・引数
SRC: ホスト側のファイルのパス
PODNAME: Podの名前
DEST: Pod内のファイルのパス

**▼Pod内のファイルをホスト側にコピー**
基本構文:

```bash
kubectl cp PODNAME:SRC DEST
```

・引数
PODNAME: Podの名前
SRC: Pod内のファイルのパス
DEST: ホスト側のファイルのパス

**📖 演習内容**:

```txt
1. `CentOS` のコンテナを含むマニフェストファイルを作成
2. `CentOS` のPodを起動
3. `sample.txt` を、作成した `CentOS` コンテナ内の `/var/tmp` ディレクトリへ転送
4. ログイン中のコンテナ内でファイルを作成して、ホスト側にコピー
5. 作業後のPodを削除
```

1.まずは任意のテキストエディタからSSHで接続し、マニフェストファイルを作成します。  
※前段の演習で使用した `tutorial` ディレクトリを使いまわし、その配下に `pod.yml` と `sample.txt` を作成します。

::: details tutorial/pod.yml, tutorial/sample.txt

```yaml:tutorial/pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: debug
  namespace: default
  labels:
    env: test
spec:
  containers:
    - name: debug
      image: centos:7
      command:
        - "sh"
        - "-c"
      args:
        - |
          while true
          do
            sleep ${DELAY}
          done
      env:
        - name: "DELAY"
          value: "5"
```

```txt:tutorial/sample.txt
Hello World !
```

:::

2.作成したマニフェストファイルを起動させます。  
テキストエディタのコマンドラインから、下記コマンドを実行して `tutorial` ディレクトリに移動します:

```bash
root@minikube:~# cd tutorial/
root@minikube:~/tutorial#
```

リソースの作成を実行します:

```bash
root@minikube:~/tutorial# kubectl apply -f pod.yml
pod/debug created
root@minikube:~/tutorial#
```

作成できたことを念のため確認します:

```bash
root@minikube:~/tutorial# kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
debug   1/1     Running   0          85s
root@minikube:~/tutorial#
```

3.ホスト内に作成した `sample.txt` を、作成した `CentOS` コンテナ内の `/var/tmp` ディレクトリへ転送します。  
わかりやすくするため、転送先のテキストファイル名を変更しています。

```bash
root@minikube:~/tutorial# kubectl cp ./sample.txt debug:/var/tmp/sample_transfer.txt
root@minikube:~/tutorial#
```

コンテナ内に入り、ファイルが転送されたことを確認します。

```bash
root@minikube:~/tutorial# kubectl exec -it debug -- sh
sh-4.2# ls /var/tmp/sample_transfer.txt
/var/tmp/sample_transfer.txt
sh-4.2#
```

4.ログイン中のコンテナ内でファイルを作成して、ホスト側にコピーします。  
手順としては、空ファイルを作成して `vi` コマンドで `Received Successfully!` と記載します。  
その後にコンテナから抜けて、ホスト側にコピーします。

```bash
sh-4.2# ~
sh-4.2# touch /var/tmp/sample_receive.txt
sh-4.2# ls /var/tmp/sample_receive.txt
/var/tmp/sample_receive.txt
sh-4.2# vi /var/tmp/sample_receive.txt
sh-4.2# cat /var/tmp/sample_receive.txt
Received Successfully!
sh-4.2# exit
exit
root@minikube:~/tutorial# kubectl cp debug:/var/tmp/sample_receive.txt ./sample_receive.txt
tar: Removing leading `/' from member names
root@minikube:~/tutorial# cat ./sample_receive.txt
Received Successfully!
root@minikube:~/tutorial#
```

5.最後に、作成したPodを削除します。

```bash
root@minikube:~/tutorial# kubectl delete -f pod.yml
pod "debug" deleted
root@minikube:~/tutorial#
```

---

### 状態/ログの確認

状態を確認する際は、`kubectl describe` コマンドを使用します。

**▼Podの状態を確認**
基本構文:

```bash
kubectl describe [TYPE/NAME]
```

・引数
TYPE: リソースの種類
NAME: リソースの名前

**▼ログの詳細を確認**
基本構文:

```bash
kubectl logs [TYPE/NAME] [--tail=n]
```

・引数
TYPE: リソースの種類
NAME: リソースの名前
--tail=n: ログの最後からn行を表示

**▼Podのログを確認**
基本構文:

```bash
kubectl logs [POD]
```

・引数
POD: Podの名前

**📖 演習内容**:

```txt
1. `CentOS` と `nginx` のコンテナを含むマニフェストファイルを作成し起動
2. `CentOS` と `nginx` のPodの状態を確認
3. `CentOS` に入る
4. `curl` コマンドを実行して `nginx` のコンテナにアクセス
5. `CentOS` から出る
6. `nginx` のPodのログを確認
7. `nginx` のPodを削除
```

1.`CentOS` と `nginx` のコンテナを含むマニフェストファイルを作成し起動します。  
※前段の演習で使用した `tutorial` ディレクトリを使いまわし、その配下に `pods.yml` を作成します。

::: details tutorial/pods.yml

```yaml:tutorial/pods.yml
apiVersion: v1
kind: Pod
metadata:
  name: debug
  namespace: default
spec:
  containers:
    - name: debug
      image: centos:7
      command:
        - "sh"
        - "-c"
      args:
        - |
          while true
          do
            sleep ${DELAY}
          done
      env:
        - name: "DELAY"
          value: "86400"

---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.17.2-alpine
```

:::

起動します。

```bash
root@minikube:~/tutorial# kubectl apply -f pods.yml
pod/debug created
pod/nginx created
root@minikube:~/tutorial#
```

2.作成したPodの状態を確認します。  
手始めに、 `debug` のPodの状態を確認します。

::: details kubectl describe pod/debug

```bash
root@minikube:~/tutorial# kubectl describe pod/debug
Name:             debug
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Fri, 30 May 2025 12:45:04 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.26
IPs:
  IP:  10.244.0.26
Containers:
  debug:
    Container ID:  docker://c0da926ebe2fc2b33718b24d2edf01480554e005fd0f2a3b770ede18b6641110
    Image:         centos:7
    Image ID:      docker-pullable://centos@sha256:be65f488b7764ad3638f236b7b515b3678369a5124c47b8d32916d6487418ea4
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
    Args:
      while true
      do
        sleep ${DELAY}
      done

    State:          Running
      Started:      Fri, 30 May 2025 12:45:05 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      DELAY:  86400
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bfm2w (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-bfm2w:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  115s  default-scheduler  Successfully assigned default/debug to minikube
  Normal  Pulled     115s  kubelet            Container image "centos:7" already present on machine
  Normal  Created    115s  kubelet            Created container: debug
  Normal  Started    115s  kubelet            Started container debug
root@minikube:~/tutorial# nning   0          10s
root@minikube:~/
```

:::

`Events` の部分に注目すると、Podの作成・起動・終了などのイベントが記載されていることがわかると思います。  
次に、 `nginx` のPodの状態を確認します。

::: details kubectl describe pod/nginx

```bash
root@minikube:~/tutorial# kubectl describe pod/nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Fri, 30 May 2025 12:45:04 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.25
IPs:
  IP:  10.244.0.25
Containers:
  nginx:
    Container ID:   docker://d8de6b8d4ccfce54ac151c3480204162fc57458c4075a699a456b522d2d00cdb
    Image:          nginx:1.17.2-alpine
    Image ID:       docker-pullable://nginx@sha256:482ead44b2203fa32b3390abdaf97cbdc8ad15c07fb03a3e68d7c35a19ad7595
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 30 May 2025 12:45:05 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v2j8z (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-v2j8z:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m15s  default-scheduler  Successfully assigned default/nginx to minikube
  Normal  Pulled     4m15s  kubelet            Container image "nginx:1.17.2-alpine" already present on machine
  Normal  Created    4m15s  kubelet            Created container: nginx
  Normal  Started    4m15s  kubelet            Started container nginx
root@minikube:~/tutorial#
```

:::

`debug` のPodと同様に、`nginx` のPodの状態も確認できました。

3.`CentOS` のPodに接続して `nginx` へアクセスしてアクセスログを見てみます。  
その前に、作成したコンテナのIPアドレスを確認します。

```bash
root@minikube:~/tutorial# kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
debug   1/1     Running   0          8m57s   10.244.0.26   minikube   <none>           <none>
nginx   1/1     Running   0          8m57s   10.244.0.25   minikube   <none>           <none>
root@minikube:~/tutorial#
```

本題の`CentOS` のPodに接続して `nginx` へアクセスしてアクセスログを見てみます。

```bash
root@minikube:~/tutorial# kubectl exec -it debug -- sh
sh-4.2#
```

`nginx` へアクセスしてアクセスログを見てみます。

::: details curl実行結果

```bash
sh-4.2# curl 10.244.0.25
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
sh-4.2#
```

:::

アクセス成功です！

5.6. コンテナから出て、`nginx` のPodの状態を確認します。

```bash
sh-4.2# exit
exit
root@minikube:~/tutorial# kubectl logs pod/nginx
10.244.0.26 - - [30/May/2025:12:55:58 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
root@minikube:~/tutorial#
```

`nginx` のPodのアクセスログが確認できました。

7.Podを削除します。

```bash
root@minikube:~/tutorial# kubectl delete -f pods.yml
pod "debug" deleted
pod "nginx" deleted
root@minikube:~/tutorial#
```

## 📕 Kubernetesリソース

リソースについては 「📋 Kubernetesリソースの理解」 の章でも記載しましたが、改めて詳細の説明を記載したいと思います。

### Pod

`Pod` とは、Kubernetesの最小単位のリソースです。  
詳細については、以下の通りになります。

- Dockerコンテナの集合体
- デプロイメントやステートフルセットなどのリソースの最小単位
- Pod内のコンテナのライフサイクルを管理する
- 1つ以上のコンテナを含むことができる
- ネットワークとストレージを共有する
- ノード間で移動することができる

下記 pod.yml の例を見てみましょう。  
::: details pod.yml

```yaml:pod.yml
apiVersion: v1
kind: Pod

〜省略〜

spec:
  containers:
  - name: nginx
  image: nginx:1.17.2-alpine
  imaagePullPolicy: never
  command: ["sh", "-c"]
  args:
  - |
    echo ${MESSAGE}
  env: [{name: MESSAGE, value: "Hello World!"}]
    volumeMounts:
    - name: storage
      mountPath: /home/nginx
  volumes:
  - name: storage
    hostPath:
      path: "/data/storage"
      type: Directory
```

:::

マニフェストファイルの書き方については以下の通りになり、主要な spec は `containers` と `volumes` です。  
例題について細かく分解して見ていきたいと思います。

**※下記はあくまで一例であり、調べてみた結果他にも種類がありました。**  
**その他の種類については公式ドキュメントを参照してください。**  
[Kubernetes公式ドキュメント：containers(英語)](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)  
[Kubernetes公式ドキュメント：volumes](https://kubernetes.io/ja/docs/concepts/storage/volumes/)

**containers**:

- `spec.containers.name`：コンテナ名を指定
- `spec.containers.image`：コンテナイメージを指定
- `spec.containers.imagePullPolicy`：コンテナイメージの取得方法を指定
  - `always` は常にイメージを取得する
  - `never` はイメージを取得しない
  - `ifNotPresent` はローカルにイメージがない場合のみ取得する
- `spec.containers.command`：コンテナ起動時に実行するコマンドを指定
- `spec.containers.args`：コンテナ起動時に実行するコマンドの引数を指定
- `spec.containers.env`：コンテナ内で使用する環境変数を指定
- `spec.containers.volumeMounts`：コンテナ内で使用するボリュームを指定
- `spec.volumes`：コンテナ内で使用するボリュームを指定
- `spec.volumes.name`：ボリューム名を指定
- `spec.volumes.mountPath`：ボリュームをマウントするパスを指定

**volumes**:

- `spec.volumes.name`：ボリューム名を指定
- `spec.volumes.hostPath`：ボリュームのパスを指定
- `spec.volumes.hostPath.path`：ボリュームのパスを指定
- `spec.volumes.hostPath.type`：ボリュームのタイプを指定

#### 演習

```txt
1. ホストにフォルダとファイルを作成
2. 作成したフォルダをマウントしたPodマニフェストファイルを作成
3. リソース作成
```

1.まずはKubernetesのホストにフォルダとファイルを作成します。

```bash
root@minikube:~# mkdir /data/strage
root@minikube:~# ls /data/
strage
root@minikube:~#
```

2.作成した `/data/strage` をマウントするようなPodマニフェストファイルを作成します。  
作成対象のファイルを格納するディレクトリは、いつも通り `tutorial` ディレクトリにします。

::: details tutorial/pod.yml

```yaml:tutorial/pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: sample
spec:
  containers:
  - name: nginx
    image: nginx:1.17.2-alpine
    volumeMounts:
    - name: storage
      mountPath: /home/nginx
  volumes:
  - name: storage
    hostPath:
      path: "/data/storage"
      type: Directory
```

:::

リソース作成前に、1.で作成したディレクトリ直下にファイルを作成して、 `/home/nginx` から作成したファイルを参照できるか確認してみます。  
対象のディレクトリ `/data/storage` に移動し、 配下に `message` ファイルを作成し、内容は `Hello World!` とします。

```bash
root@minikube:~# cd /data/storage/
root@minikube:/data/storage# vi message
root@minikube:/data/storage# cat message
Hello World!
root@minikube:/data/storage#
```

3.リソースを作成し、作成したPodに接続してファイルの内容を確認します。  
まずは `tutorial` ディレクトリに移動し、リソースを作成してコンテナ内に接続します。

```bash
root@minikube:~# cd tutorial/
root@minikube:~/tutorial# kubectl apply -f pod.yml
pod/sample created
root@minikube:~/tutorial# kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
sample   1/1     Running   0          13s
root@minikube:~/tutorial# kubectl exec -it sample -- sh
/ #
```

`/home/nginx` にマウントしたディレクトリに移動し、ファイルの内容を確認します。

```bash
/ # cd /home/nginx/
/home/nginx # ls
message
/home/nginx # cat message
HEllo World!
/home/nginx #
```

ファイルの内容が確認できました。  
演習としては完了なので、コンテナから抜けてPodを削除します。

---

### ReplicaSet

`ReplicaSet` とは、Podのレプリカを管理するためのリソースです。  
詳細については、以下の通りになります。

- Podのレプリカを管理する
- Podの数を指定することができる
- Podのスケールアウト・スケールインを自動で行う
- Podのステータスを監視する
- Podのステータスが変化した場合に、指定した数のPodを自動で起動する
  - 例えば、Podが異常終了した場合に、指定した数のPodを自動で起動する

下記 replicaset.yml の例を見てみましょう。  
:::details tutorial/replicaset.yml

```yaml:replicaset.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
      env: test
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: test
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2-alpine
```

:::

マニフェストファイルの書き方については以下の通りになり、主要な spec は `replicas` 、 `selector` 、 `template` です。  
例題について細かく分解して見ていきたいと思います。  
[Kubernetes公式ドキュメント：ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

**replicas**:

- `spec.replicas`：Podを複製する数を指定

**selector**:

- `spec.selector`：Podのラベルを指定

**template**:

- `spec.template`：Podのテンプレートを指定
- `spec.template.metadata`：Podのメタデータを指定
- `spec.template.metadata.name`：Podの名前を指定
- `spec.template.metadata.labels`：Podのラベルを指定
- `spec.template.spec`：Podの仕様を指定
- `spec.template.spec.containers`：Pod内のコンテナを指定
- `spec.template.spec.containers.name`：コンテナ名を指定
- `spec.template.spec.containers.image`：コンテナイメージを指定

#### 演習

```txt
1. ReplicaSetマニフェストファイルを作成
2. リソース作成
3. 手動でスケールアウト(構築するサーバーの台数を増やす)
```

1.まずは `tutorial` ディレクトリに移動し、ReplicaSetマニフェストファイルを作成します。
::: details tutorial/replicaset.yml

```yaml:tutorial/replicaset.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
      env: test
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: test
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2-alpine
```

:::

2.リソースを作成します。

```bash
root@minikube:~/tutorial# kubectl apply -f replicaset.yml
replicaset.apps/nginx created
root@minikube:~/tutorial#
```

出力結果から、3つのPodが作成されていることが確認できます。  
また、 `kubectl get all` コマンドですべてのリソースの状態を確認することができます。

```bash
root@minikube:~/tutorial# kubectl get all
NAME              READY   STATUS    RESTARTS   AGE
pod/nginx-mp4bl   1/1     Running   0          60m
pod/nginx-rnfsx   1/1     Running   0          60m
pod/nginx-wl77v   1/1     Running   0          60m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d2h

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx   3         3         3       60m
root@minikube:~/tutorial#
```

3.手動でスケールアウトします。  
方法としては、マニフェストファイルを編集し、 `replicas` の値を変更します。
ここでは、 `replicas` の値を `5` に変更します。

::: details tutorial/replicaset.yml

```yaml:tutorial/replicaset.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 5 # 3 -> 5 に変更
  selector:
    matchLabels:
      app: web
      env: test
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: test
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2-alpine
```

:::

変更したマニフェストファイルを適用し、リソースを再度作成します。

```bash
root@minikube:~/tutorial# kubectl apply -f replicaset.yml
replicaset.apps/nginx configured
root@minikube:~/tutorial# kubectl get all
NAME              READY   STATUS    RESTARTS   AGE
pod/nginx-9n9bs   1/1     Running   0          13s
pod/nginx-dlbw2   1/1     Running   0          13s
pod/nginx-mp4bl   1/1     Running   0          64m
pod/nginx-rnfsx   1/1     Running   0          64m
pod/nginx-wl77v   1/1     Running   0          64m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d2h

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx   5         5         5       64m
root@minikube:~/tutorial#
```

出力結果より、Podが5つになっていることが確認できます。  
このように、ReplicaSetを使うことで、Podの数を簡単に増減させることができます。  
最後に、リソースを削除して終了します。

---

### Deployment

---

### Service

---

### ConfigMap

---

### Secret

---

### 永続データ (PersistentVolume, PersistentVolumeClaim)

---

### StatefulSet

---

### Ingress

<!-- ## ※ここから続き -->

## 🎉 まとめ

長い時間ご覧いただき、ありがとうございました。
この記事では、Kubernetesの基本概念から実際のハンズオンまでを通して学習しました。

**学習した内容**:

- Kubernetesの基本概念とDockerとの違い
- Kubernetesのメリット・デメリット
- minikubeを使った環境構築
- 基本的なkubectlコマンドの使い方
- Podの作成・確認・削除の実践

Kubernetesは学習コストが高いツールですが、コンテナオーケストレーションにおいて非常に強力な機能を提供します。今回の基礎学習を土台に、さらに深い理解を目指していきたいと思います。

次回は、より実践的なリソースについて学習しアウトプットしていく予定です。
