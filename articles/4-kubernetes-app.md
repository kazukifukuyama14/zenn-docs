---
title: "ミリしらKubernetes〜ド初心者がKubernetesをある程度理解するまでの記録〜<実践編>"
emoji: "🕸️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Kubernetes, Docker]
published: true
published_at: 2025-06-04 09:00
---

## 🙇‍♂️ はじめに

この記事ではKubernetesの実践編となっており、実際に手を動かしながら学習した記録になります。

基礎編は下記記事に掲載していますので、お時間あるときにどうぞ！  
https://zenn.dev/wan0ri/articles/aa3c2efdde0b1c

## 📕 Kubernetesリソース

リソースについては 前回の [📋 Kubernetesリソースの理解](https://zenn.dev/articles/aa3c2efdde0b1c/edit#%F0%9F%93%8B-kubernetes%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AE%E7%90%86%E8%A7%A3) の章でも記載しましたが、改めて詳細の説明を記載したいと思います。

### Pod

`Pod` とは、Kubernetesの最小単位のリソースです。  
詳細については、以下の通りになります。

- Dockerコンテナの集合体
- デプロイメントやステートフルセットなどのリソースの最小単位
- Pod内のコンテナのライフサイクルを管理する
- 1つ以上のコンテナを含むことができる
- ネットワークとストレージを共有する
- ノード間で移動することができる

下記 `pod.yml` の例を見てみましょう:
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

一例として、マニフェストファイルの書き方について、主要な spec は以下の通り記載します。

**※下記はあくまで一例であり、調べてみた結果他にも種類がありました。**  
**その他の種類については公式ドキュメントを参照してください。**

https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/  
https://kubernetes.io/docs/concepts/storage/volumes/

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

```txt:お題目
1. ホストにフォルダとファイルを作成
2. 作成したフォルダをマウントしたPodマニフェストファイルを作成
3. リソース作成
```

1.まずはKubernetesのホストにフォルダとファイルを作成します:

```bash:minikube
root@minikube:~# mkdir /data/strage
root@minikube:~# ls /data/
strage
root@minikube:~#
```

2.作成した `/data/strage` をマウントするようなPodマニフェストファイルを作成します。  
作成対象のファイルを格納するディレクトリは、いつも通り `tutorial` ディレクトリにします:
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
対象のディレクトリ `/data/storage` に移動し、 配下に `message` ファイルを作成し、内容は `Hello World!` とします:

```bash:minikube
root@minikube:~# cd /data/storage/
root@minikube:/data/storage# vi message
root@minikube:/data/storage# cat message
Hello World!
root@minikube:/data/storage#
```

3.リソースを作成し、作成したPodに接続してファイルの内容を確認します。  
まずは `tutorial` ディレクトリに移動し、リソースを作成してコンテナ内に接続します:

```bash:minikube
root@minikube:~# cd tutorial/
root@minikube:~/tutorial# kubectl apply -f pod.yml
pod/sample created
root@minikube:~/tutorial# kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
sample   1/1     Running   0          13s
root@minikube:~/tutorial# kubectl exec -it sample -- sh
/ #
```

`/home/nginx` にマウントしたディレクトリに移動し、ファイルの内容を確認します:

```bash:minikube
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

下記 `replicaset.yml` の例を見てみましょう:
:::details replicaset.yml

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

一例として、マニフェストファイルの書き方について、主要な spec は以下の通り記載します。  
**※下記はあくまで一例であり、調べてみた結果他にも種類がありました。**  
**その他の種類については公式ドキュメントを参照してください。**

https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

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

```txt:お題目
1. ReplicaSetマニフェストファイルを作成
2. リソース作成
3. 手動でスケールアウト(構築するサーバーの台数を増やす)
```

1.まずは `tutorial` ディレクトリに移動し、ReplicaSetマニフェストファイルを作成します:
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

2.リソースを作成します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f replicaset.yml
replicaset.apps/nginx created
root@minikube:~/tutorial#
```

出力結果から、3つのPodが作成されていることが確認できます。  
また、 `kubectl get all` コマンドですべてのリソースの状態を確認することができます:

```bash:minikube
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
ここでは、 `replicas` の値を `5` に変更します:
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

変更したマニフェストファイルを適用し、リソースを再度作成します:

```bash:minikube
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
演習としては完了なので、コンテナから抜けてPodを削除します。

---

### Deployment

`Deployment` とは、 ReplicaSet の集合体で、それらの世代管理ができるリソースです。  
詳細については、以下の通りになります。

- Podのレプリカを管理する
- Podの数を指定することができる
- Podのスケールアウト・スケールインを自動で行う
- Podのステータスを監視する
- Podのステータスが変化した場合に、指定した数のPodを自動で起動する

下記 `deployment.yml` の例を見てみましょう:
:::details deployment.yml

```yaml:deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  annotations:
    kubernetes.io/change-cause: "First release."
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      env: study
  revisionHistoryLimit: 14
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: study
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2-alpine

```

:::

一例として、マニフェストファイルの書き方について、主要な spec は以下の通り記載します。  
**※下記はあくまで一例であり、調べてみた結果他にも種類がありました。**  
**その他の種類については公式ドキュメントを参照してください。**

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

**replicas**:

- `spec.replicas`：Podを複製する数を指定

**selector**:

- `spec.selector`：Podのラベルを指定
- `spec.selector.matchLabels`：Podのラベルを指定
- `spec.selector.matchLabels.app`：Podのラベルを指定
- `spec.selector.matchLabels.env`：Podのラベルを指定

**revisionHistoryLimit**:

- `spec.revisionHistoryLimit`：Deploymentの履歴を保持する数を指定(デフォルトは**10**)

**strategy**:
※**Kubernetes v1.15 以降では RollingUpdate がデフォルト**になりました。

- `spec.strategy`：スケーリングの方法を指定
- `spec.strategy.type`：スケーリングのタイプを指定
- `spec.strategy.rollingUpdate`：スケーリングの方法を指定
- `spec.strategy.rollingUpdate.maxSurge`：スケールアウト時に作成するPodの数を指定
  (**レプリカ数を超えてよいPod数**)
- `spec.strategy.rollingUpdate.maxUnavailable`：スケールイン時に削除するPodの数を指定
  (**一度に消失してよいPod数**)

ロールアウト履歴を表示するコマンドは以下の通りです:

```bash:minikube
kubectl rollout history [TYPE/NAME] --to-revision=N
```

・引数
`TYPE`:リソースの種類  
`NAME`:リソース名  
`--to-revision`:ロールアウト履歴の番号(デフォルトは0)

**template**:

- `spec.template.metadata.labels`：Podのラベルを指定
- `spec.template.spec.containers.name`：コンテナ名を指定
- `spec.template.spec.containers.image`：コンテナイメージを指定

#### 演習

```txt:お題目
1. Deploymentマニフェストファイルを作成
2. リソースを作成
3. ロールアウト履歴確認
4. Deployment修正
5. ロールアウト履歴確認
6. ロールバック
```

1. Deploymentマニフェストファイルを作成します:

::: details tutorial/deployment.yml

```yaml:tutorial/deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      env: study
  revisionHistoryLimit: 14
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: study
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2-alpine
```

:::

2.リソースを作成します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f deployment.yml
deployment.apps/nginx created
root@minikube:~/tutorial# kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-7f5db5465d-s9tf9   1/1     Running   0          17s
pod/nginx-7f5db5465d-xhfzl   1/1     Running   0          17s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d17h

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           17s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-7f5db5465d   2         2         2       17s
root@minikube:~/tutorial#
```

5.ロールアウト履歴を確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl rollout history deployment/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>

root@minikube:~/tutorial#
```

実行結果から、ロールアウト履歴は1つしかありません。  
出力結果の `CHANGE-CAUSE` の部分は、デフォルトでは空になっています。

4.Deployment修正します。
こちらを更新するには、マニフェストファイルの `annotations` に `kubernetes.io/change-cause` を追加してメッセージを記載します:
:::details tutorial/deployment.yml

```yaml:tutorial/deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  annotations: # 追加
    kubernetes.io/change-cause: "Update nginx" # 追加
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      env: study
  revisionHistoryLimit: 14
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: study
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.3-alpine # バージョンを変更

```

:::
再度リソースを作成して、ロールアウト履歴を確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f deployment.yml
deployment.apps/nginx configured
root@minikube:~/tutorial# kubectl rollout history deploy/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         Update nginx

root@minikube:~/tutorial#
```

annotations に記載したメッセージが、ロールアウト履歴に表示されました。  
しかし、nginxのバージョンがUpした履歴が出力されていません。  
これは、デフォルトでは、ロールアウト履歴にリソースの変更内容が表示されないためです。  
それに対しては、リビジョンを指定することで確認できます:

```bash:minikube
root@minikube:~/tutorial# kubectl rollout history deploy/nginx --revision=2
deployment.apps/nginx with revision #2
Pod Template:
  Labels:       app=web
        env=study
        pod-template-hash=9fc9b8565
  Annotations:  kubernetes.io/change-cause: Update nginx
  Containers:
   nginx:
    Image:      nginx:1.17.3-alpine
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors:       <none>
  Tolerations:  <none>

root@minikube:~/tutorial#
```

`kubectl rollout history` コマンドの基本出力では、**change-cause のみが表示**されます。  
具体的なイメージバージョンなどの詳細情報を確認するには、**--revision オプションで特定のリビジョンを指定する**必要があります。

6.直前のバージョンに戻すには、以下のコマンドを実行します:

```bash:minikube
root@minikube:~/tutorial# kubectl rollout undo deployment/nginx
deployment.apps/nginx rolled back
root@minikube:~/tutorial# kubectl rollout history deploy/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
2         Update nginx
3         <none>

root@minikube:~/tutorial#
```

REVISION 3 が追加され、ロールバックされました。  
このように、ロールバックは `kubectl rollout undo` コマンドで実行できます。  
演習としては完了なので、コンテナから抜けてPodを削除します。

---

### Service

`Service` は、外部公開、内部通信、名前解決などの機能を提供します。  
種別としては、以下の種類があります。

- **ClusterIP**
  - クラスタネットワーク内にIPアドレスを公開
  - 名前設定でPodへ到達できるようにする
- **NodePort**
  ClusterIP に加え、Node のポートにマッピングして受け付けられるようにする
- **LoadBalancer**
  NodePort に加え、クラウドプロバイダのロードバランサーを利用してサービスを公開する
- **ExternalName**
  外部サービスに接続

下記 `service.yml` の例を見てみましょう:

::: details service.yml

```yaml:service.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: web
    env: study
spec:
  containers:
  - name: nginx
    image: nginx:1.17.2-alpine

---
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  type: ClusterIP
  clusterIP: 10.101.10.100
  selector:
    app: web
    env: study
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30000
```

:::

一例として、マニフェストファイルの書き方について、主要な spec は以下の通り記載します。  
**※下記はあくまで一例であり、調べてみた結果他にも種類がありました。**  
**その他の種類については公式ドキュメントを参照してください。**

https://kubernetes.io/docs/concepts/services-networking/service/

**containers**:

- `spec.containers`: コンテナの設定
  - `spec.containers.image`: コンテナイメージ
- `spec.type`: サービスの種類
- `spec.clusterIP`: クラスターIP
  - `spec.clusterIP` を指定しない場合、自動的にクラスターIPが割り当てられる
  - `spec.clusterIP` を指定した場合、指定したクラスターIPが割り当てられる
- `spec.selector`: サービスのラベル
- `spec.ports`: サービスのポート
  - `spec.ports.port`: サービスのポート
  - `spec.ports.targetPort`: コンテナ転送先ポート
  - `spec.ports.nodePort`: Nodeのポート

#### 演習

```txt:お題目
1.NodePortのServiceマニフェストファイルを作成
2.リソース作成
3.ブラウザへアクセスして動作確認
```

1.NodePortのServiceマニフェストファイルを作成します:
::: details tutorial/service.yml

```yaml:tutorial/service.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: web
    env: study
spec:
  containers:
  - name: nginx
    image: nginx:1.17.2-alpine
    ports:
    - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  type: NodePort
  selector:
    app: web
    env: study
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30000
```

:::

2.リソースを作成します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f service.yml
pod/nginx created
service/web-svc created
root@minikube:~/tutorial#
```

```bash:minikube
root@minikube:~/tutorial# kubectl get all
NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          11s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        6d21h
service/web-svc      NodePort    10.102.57.173   <none>        80:30000/TCP   11s
root@minikube:~/tutorial#
```

`web-svc` リソースが作成されました。

3.ブラウザへアクセスして動作確認します。
前段で作成した service.yml に記載の通り、80番ポートから NodePort の30000番ポートにアクセスします。  
やり方としては、仮想マシンのIPアドレスのあとに `:30000` を付与します。  
自端末のターミナルから、以下コマンドを実行すると仮想マシンのIPアドレスを確認できます。

```bash:terminal
minikube ip
192.168.49.2
```

上のコマンド結果から、`192.168.49.2:30000` となります。

![access-loop](/images/kubernetes-tutorial/access-loop.png)

アクセスできないようですので、原因の確認をします。  
確実な方法として、ポートフォワーディングでの確認を自端末から実施します:

```bash:terminal
kubectl port-forward service/web-svc 8080:80
```

ブラウザで <http://localhost:8080> にアクセス。

![port-forward](/images/kubernetes-tutorial/port-forward.png)

ひとまず表示されたようです。

次に、 `minikube service web-avc` コマンドを実行して、ブラウザでアクセスします:

```bash:terminal
minikube service web-svc

|-----------|---------|-------------|---------------------------|
| NAMESPACE |  NAME   | TARGET PORT |            URL            |
|-----------|---------|-------------|---------------------------|
| default   | web-svc |          80 | http://192.168.49.2:30000 |
|-----------|---------|-------------|---------------------------|
🏃  web-svc サービス用のトンネルを起動しています。
|-----------|---------|-------------|------------------------|
| NAMESPACE |  NAME   | TARGET PORT |          URL           |
|-----------|---------|-------------|------------------------|
| default   | web-svc |             | http://127.0.0.1:51389 |
|-----------|---------|-------------|------------------------|
🎉  デフォルトブラウザーで default/web-svc サービスを開いています...
❗  Docker ドライバーを darwin 上で使用しているため、実行するにはターミナルを開く必要があります。
```

これだとアクセスできるようですが、転送先のポート番号が変わっています:

::: details 30000ポート調査

```bash:terminal
sudo lsof -i -P -n | grep LISTEN

ControlCe   598         wan0ri    8u  IPv4  0x3afdecce503bcbc      0t0    TCP *:7000 (LISTEN)
ControlCe   598         wan0ri    9u  IPv6 0x99f4c63a72bbee16      0t0    TCP *:7000 (LISTEN)
ControlCe   598         wan0ri   10u  IPv4 0xff7d2e786c2f4154      0t0    TCP *:5000 (LISTEN)
ControlCe   598         wan0ri   11u  IPv6 0x7b0ab8d631810a8f      0t0    TCP *:5000 (LISTEN)
rapportd    647         wan0ri    8u  IPv4 0xe787816e3568c0c0      0t0    TCP *:49152 (LISTEN)
rapportd    647         wan0ri    9u  IPv6 0xe16e33c168b68887      0t0    TCP *:49152 (LISTEN)
Raycast     683         wan0ri   42u  IPv4 0x66633fceb00a43dc      0t0    TCP 127.0.0.1:7265 (LISTEN)
logioptio   739         wan0ri   40u  IPv4 0xf6c87394e39caa96      0t0    TCP *:59869 (LISTEN)
Stream      748         wan0ri   21u  IPv4 0x1584f5c902f4fc3e      0t0    TCP 127.0.0.1:28196 (LISTEN)
Stream      748         wan0ri   83u  IPv6 0x2029c113012954bd      0t0    TCP *:28198 (LISTEN)
Google      825         wan0ri   43u  IPv6 0x785b2916e715207f      0t0    TCP [::1]:7679 (LISTEN)
Cursor     1834         wan0ri   28u  IPv4  0xf1e43c229680aa6      0t0    TCP 127.0.0.1:49656 (LISTEN)
Cursor     1836         wan0ri  101u  IPv6 0x5396a379897848a3      0t0    TCP *:49900 (LISTEN)
com.docke  3641         wan0ri  142u  IPv4 0xd0d37d36cd1fe794      0t0    TCP 127.0.0.1:49647 (LISTEN)
com.docke  3641         wan0ri  164u  IPv4 0x7beb3108c1e8146b      0t0    TCP 127.0.0.1:49648 (LISTEN)
com.docke  3641         wan0ri  165u  IPv4 0xa7501b375430a281      0t0    TCP 127.0.0.1:49649 (LISTEN)
com.docke  3641         wan0ri  166u  IPv4 0x4422138d50fce971      0t0    TCP 127.0.0.1:49650 (LISTEN)
com.docke  3641         wan0ri  175u  IPv4 0xc75cc9a524e61117      0t0    TCP 127.0.0.1:49651 (LISTEN)
com.docke  3641         wan0ri  186u  IPv4 0xcdfa1e7b642cea8b      0t0    TCP 127.0.0.1:6443 (LISTEN)
ssh        4848         wan0ri    5u  IPv6 0xdb9cf91b4ecd09a4      0t0    TCP [::1]:50720 (LISTEN)
ssh        4848         wan0ri    6u  IPv4 0x281c149607849139      0t0    TCP 127.0.0.1:50720 (LISTEN)
Cursor     5389         wan0ri   35u  IPv4 0x8a7d5f1734985fb2      0t0    TCP 127.0.0.1:46329 (LISTEN)
node       5657         wan0ri   14u  IPv6 0x2889e17cd884fa2c      0t0    TCP *:49901 (LISTEN)
kubectl    6078         wan0ri    8u  IPv4 0xa3c01d69d3e2afda      0t0    TCP 127.0.0.1:8080 (LISTEN)
kubectl    6078         wan0ri    9u  IPv6 0x1183030740956343      0t0    TCP [::1]:8080 (LISTEN)
ssh        9893         wan0ri    5u  IPv6 0x6a940d1f8ec2aa14      0t0    TCP [::1]:51731 (LISTEN)
ssh        9893         wan0ri    6u  IPv4 0x857aab088d9057f4      0t0    TCP 127.0.0.1:51731 (LISTEN)

sudo lsof -i :30000

netstat -an | grep LISTEN

tcp4       0      0  127.0.0.1.51731        *.*                    LISTEN
tcp6       0      0  ::1.51731              *.*                    LISTEN
tcp6       0      0  ::1.8080               *.*                    LISTEN
tcp4       0      0  127.0.0.1.8080         *.*                    LISTEN
tcp4       0      0  127.0.0.1.50720        *.*                    LISTEN
tcp6       0      0  ::1.50720              *.*                    LISTEN
tcp4       0      0  127.0.0.1.49656        *.*                    LISTEN
tcp46      0      0  *.28198                *.*                    LISTEN
tcp46      0      0  *.49901                *.*                    LISTEN
tcp46      0      0  *.49900                *.*                    LISTEN
tcp4       0      0  127.0.0.1.46329        *.*                    LISTEN
tcp4       0      0  127.0.0.1.49651        *.*                    LISTEN
tcp4       0      0  127.0.0.1.49650        *.*                    LISTEN
tcp4       0      0  127.0.0.1.49649        *.*                    LISTEN
tcp4       0      0  127.0.0.1.49648        *.*                    LISTEN
tcp4       0      0  127.0.0.1.49647        *.*                    LISTEN
tcp4       0      0  127.0.0.1.6443         *.*                    LISTEN
tcp4       0      0  *.59869                *.*                    LISTEN
tcp4       0      0  127.0.0.1.28196        *.*                    LISTEN
tcp6       0      0  ::1.7679               *.*                    LISTEN
tcp4       0      0  127.0.0.1.7265         *.*                    LISTEN
tcp6       0      0  *.49152                *.*                    LISTEN
tcp4       0      0  *.49152                *.*                    LISTEN
tcp6       0      0  *.5000                 *.*                    LISTEN
tcp4       0      0  *.5000                 *.*                    LISTEN
tcp6       0      0  *.7000                 *.*                    LISTEN
tcp4       0      0  *.7000                 *.*                    LISTEN

sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
Firewall is enabled. (State = 1)

sudo /usr/libexec/ApplicationFirewall/socketfilterfw --listapps
Total number of apps = 15
1 : /usr/libexec/dhcp6d
             (Allow incoming connections)
2 : /Library/Application Support/Logitech.localized/LogiOptionsPlus/logioptionsplus_agent.app/Contents/MacOS/logioptionsplus_agent
             (Allow incoming connections)
3 : /System/Library/CoreServices/UniversalControl.app/Contents/MacOS/UniversalControl
             (Allow incoming connections)
4 : /System/Library/CoreServices/ControlCenter.app/Contents/MacOS/ControlCenter
             (Allow incoming connections)
5 : /System/Library/PrivateFrameworks/ReplicatorCore.framework/Support/replicatord
             (Allow incoming connections)
6 : /Applications/Google Chrome.app
             (Allow incoming connections)
7 : /Applications/Spotify.app
             (Allow incoming connections)
8 : /usr/libexec/rapportd
             (Allow incoming connections)
9 : /usr/libexec/remoted
             (Allow incoming connections)
10 : /usr/bin/python3
             (Allow incoming connections)
11 : /usr/bin/ruby
             (Allow incoming connections)
12 : /usr/sbin/cupsd
             (Allow incoming connections)
13 : /usr/libexec/sharingd
             (Allow incoming connections)
14 : /usr/libexec/sshd-keygen-wrapper
             (Allow incoming connections)
15 : /usr/sbin/smbd
             (Allow incoming connections)
```

:::

そもそも、30000ポートはブロックされているわけではなく、**そのポートでリッスンしているプロセスが存在しない**ことがわかりました。

- ✅ ファイアウォールは有効だが、30000番ポートをブロックしていない
- ✅ 8080番ポート（kubectl port-forward）は正常にリッスン中
- ❌ 30000番ポートでリッスンしているプロセスが存在しない

このことから、マニフェストファイルで割り当てている30000ポートを修正し、動的にポートを割り当てるように修正します:

::: details マニフェストファイル修正

```yaml:tutorial/service.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: web
    env: study
spec:
  containers:
  - name: nginx
    image: nginx:1.17.2-alpine
    ports:
    - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  type: NodePort
  selector:
    app: web
    env: study
  ports:
  - port: 80
    targetPort: 80
    # nodePort: 30000 = 自動割り当てのため、明示的に指定しない
```

:::

再度リソースを作成し、実際に割り当てられたポートを確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f service.yml
pod/nginx created
service/web-svc created
root@minikube:~/tutorial# kubectl get service web-svc
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
web-svc   NodePort   10.105.161.23   <none>        80:30410/TCP   9s
root@minikube:~/tutorial#
```

次に、自端末からアクセスコマンドを実行し、ブラウザで確認してみます:

```bash:terminal
minikube service web-svc
|-----------|---------|-------------|---------------------------|
| NAMESPACE |  NAME   | TARGET PORT |            URL            |
|-----------|---------|-------------|---------------------------|
| default   | web-svc |          80 | http://192.168.49.2:30410 |
|-----------|---------|-------------|---------------------------|
🏃  web-svc サービス用のトンネルを起動しています。
|-----------|---------|-------------|------------------------|
| NAMESPACE |  NAME   | TARGET PORT |          URL           |
|-----------|---------|-------------|------------------------|
| default   | web-svc |             | http://127.0.0.1:53016 |
|-----------|---------|-------------|------------------------|
🎉  デフォルトブラウザーで default/web-svc サービスを開いています...
❗  Docker ドライバーを darwin 上で使用しているため、実行するにはターミナルを開く必要があります。
```

![access-retry](/images/kubernetes-tutorial/access-retry.png)

無事にアクセスできました！🎉

演習としては完了なので、コンテナから抜けてPodを削除します。

### ConfigMap

Kubernetesでは、設定情報を管理するためのリソースとしてConfigMapがあります。  
ConfigMapを使用することで、以下のようなメリットがあります。

- 設定情報をコンテナイメージから切り離し、より柔軟な管理が可能になる
- 設定情報をコンテナイメージから切り離すことで、アプリケーションの再ビルドや再デプロイを必要とせずに設定情報を変更できる

下記 `configmap.yml` の例を見てみましょう:

::: details configmap.yml

```yaml:configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sample-config
  namespace: default
data:
  sample.cfg: |
    user: taro.tanaka
  type: "application"

---
apiVersion: v1
kind: Pod
metadata:
  name: sample
  namespace: default
spec:
  containers:
  - name: sample
    image: nginx:1.17.2-alpine
    env:
    - name: TYPE
      valueFrom:
        configMapKeyRef:
          name: sample-config
          key: type
    volumeMounts:
    - name: config-storage
      mountPath: /home/nginx
  volumes:
  - name: config-storage
    configMap:
      name: sample-config
      items:
      - key: sample.cfg
        path: sample.cfg
```

:::

一例として、マニフェストファイルの書き方については、これまでは spec に着目していましたが ConfigMap では **data にキーバリューで保存**します。  
**※下記はあくまで一例であり、調べてみた結果他にも種類がありました。**  
**その他の種類については公式ドキュメントを参照してください。**

https://kubernetes.io/docs/concepts/configuration/configmap/

**data**:

- `data`: 設定情報を保存するためのキーバリュー形式のデータ
- `data.sample.cfg`: ファイル形式の設定データ（複数行のテキスト）
- `data.type`: キー・バリュー形式の設定データ（単一の値）

**containers**:

- `spec.containers.env.valueForm`: 値の取得元を指定
- `spec.containers.env.valueFrom.configMapKeyRef.name`: ConfigMapから値を参照
- `spec.containers.env.valueFrom.configMapKeyRef.key`: ConfigMapのキーを指定

リソースの利用方法は、2種類あります。

| 利用方法               | ポイント                                            |
| ---------------------- | --------------------------------------------------- |
| ワークロ環境変数へ渡す | spec.containers.env.valueForm に ConfigMap を指定   |
| ファイルとしてマウント | spec.volumes と spec.containers.volumeMounts に指定 |

#### 演習

```txt:お題目
1.ConfigMap と Pod を含むマニフェストファイルを作成
2.リソースを作成
3.Pod に入って ConfigMap が接続されていることを確認
```

1. ConfigMap と Pod を含むマニフェストファイルを作成します:

::: details tutorial/configmap.yml

```yaml:tutorial/configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sample-config
  namespace: default
data:
  sample.cfg: |
    user: taro.tanaka
  type: "application"

---
apiVersion: v1
kind: Pod
metadata:
  name: sample
  namespace: default
spec:
  containers:
  - name: sample
    image: nginx:1.17.2-alpine
    env:
    - name: TYPE
      valueFrom:
        configMapKeyRef:
          name: sample-config
          key: type
    volumeMounts:
    - name: config-storage
      mountPath: /home/nginx
  volumes:
  - name: config-storage
    configMap:
      name: sample-config
      items:
      - key: sample.cfg
        path: sample.cfg
```

:::

2.リソースを作成します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f configmap.yml
configmap/sample-config created
pod/sample created
root@minikube:~/tutorial#
```

3.マウント先の `/home/nginx` に `sample.cfg` が存在することを確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl exec -it sample -- shs
/ # ls /home/nginx/
sample.cfg
/ # cat /home/nginx/sample.cfg
user: taro.tanaka
/ #
```

`sample.cfg` の存在を確認でき、かつファイルの内容も確認できました。  
このように、ConfigMap を使用することで、設定情報をコンテナイメージから切り離し、より柔軟な管理が可能になります。  
演習としては完了なので、コンテナから抜けてPodを削除します。

---

### Secret

Kubernetes上で機密情報を管理するためのリソースとしてSecretがあります。
Secretを使用することで、以下のようなメリットがあります。

- 機密情報をコンテナイメージから切り離し、より柔軟な管理が可能になる
- 機密情報をコンテナイメージから切り離すことで、アプリケーションの再ビルドや再デプロイを必要とせずに機密情報を変更できる
- コンテナイメージに機密情報を含めることを避けることができる
- Kubernetesのコンポーネント間で機密情報を共有することができる
- 機密情報を暗号化して保存することで、セキュリティを向上させることができる

下記 `secret.yml` の例を見てみましょう:
::: details secret.yml

```yaml:secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: sample-secret
data:
  message: SGVsbG8gV29ybGQgIQ==     # echo -n 'Hello World !' | base64
  keyfile: WU9VUi1TRUNSRVQtS0VZ     # cat ./keyfile | base64

---
apiVersion: v1
kind: Pod
metadata:
  name: sample
  namespace: default
spec:
  containers:
  - name: sample
    image: nginx:1.17.2-alpine
    env:
    - name: MESSAGE
      valueFrom:
        secretKeyRef:
          name: sample-secret
          key: message
    volumeMounts:
    - name: secret-storage
      mountPath: /home/nginx
  volumes:
  - name: secret-storage
    secret:
      secretName: sample-secret
      items:
      - key: keyfile
        path: keyfile
```

:::

一例として、マニフェストファイルの書き方についてはSecretでは **data にキーバリューで保存**します。  
**※下記はあくまで一例であり、調べてみた結果他にも種類がありました。**  
**その他の種類については公式ドキュメントを参照してください。**

https://kubernetes.io/docs/concepts/configuration/secret/

**data**:

- `data`: 機密情報を保存するためのキーバリュー形式のデータ
- `data.message`: メッセージを保存するためのキーバリュー形式のデータ
- `data.keyfile`: キーファイルを保存するためのキーバリュー形式のデータ
  **※base64エンコードされています。**  
  base64エンコードについては、[base64エンコード・デコード](https://qiita.com/kazukimatsumoto/items/13170a85242449140121)を参照してください。

**containers**:

- `spec.containers.env.valueForm`: 値の取得元を指定
- `spec.containers.env.valueFrom.secretKeyRef.name`: Secretから値を参照
- `spec.containers.env.valueFrom.secretKeyRef.key`: Secretのキーを指定
- `spec.containers.volumeMounts`: マウント先のパスを指定
- `spec.volumes`: マウント先のパスを指定
- `spec.volumes.secret.secretName`: Secretの名前を指定
- `spec.volumes.secret.items`: マウント先のパスを指定
- `spec.volumes.secret.items.key`: マウント先のパスを指定
- `spec.volumes.secret.items.path`: マウント先のパスを指定

リソース生成方法は、2種類あります。

| 利用方法                     | ポイント                                                                    |
| ---------------------------- | --------------------------------------------------------------------------- |
| コマンドで直接生成           | キーバリューは複数で指定                                                    |
| マニフェストファイルから生成 | マニフェストファイルから生成は `kubectl apply` / Base64文字列の値取得が必要 |

**コマンドで直接生成**:

```bash:minikube
kubectl create secret generic NAME OPTION
```

・引数  
`NAME`: リソース名  
`OPTION`: オプション

・オプション  
`--from-literal`: キーバリュー形式のデータを指定  
`--from-file`: ファイルからキーバリュー形式のデータを指定  
`--from-env-file`: 環境変数ファイルからキーバリュー形式のデータを指定

**コマンドでBase64文字列の値を取得**:

```bash:minikube
echo -n 'TEXT' | base64
```

・引数
`TEXT`: Base64変換したい文字列

リソース利用方法は、2種類あります。

| 利用方法               | ポイント                                                      |
| ---------------------- | ------------------------------------------------------------- |
| 環境変数へ渡す         | spec.containers.env.valueFrom に Secret を指定                |
| ファイルとしてマウント | spec.volumes と spec.containers.volumeMounts に Secret に指定 |

#### 演習

```txt:お題目
1.Secret と Pod を含むマニフェストファイルを作成
2.リソースを作成
3.Pod に入って Secret が接続されていることを確認
```

1.まず初めに、コマンドから Secret を作成します。  
「リテラル」と「ファイル」の2種類で `tutorial` 配下にSecret を作成します:

```bash:keyfile
OUR-SECRET-KEY
```

```bash:minikube
root@minikube:~/tutorial# ls
keyfile  secret.yml
root@minikube:~/tutorial#
```

コマンドを実行します:

```bash:minikube
kubectl create secret generic sample-secret \
--from-literal=message='Hello World!' \
--from-file=keyfile=./keyfile
```

作成されているか確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl get secret
NAME            TYPE     DATA   AGE
sample-secret   Opaque   2      44s
root@minikube:~/tutorial#
```

中身の確認をYAML形式で確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl get secret/sample-secret -o yaml
apiVersion: v1
data:
  keyfile: T1VSLVNFQ1JFVC1LRVk=
  message: SGVsbG8gV29ybGQh
kind: Secret
metadata:
  creationTimestamp: "2025-06-01T09:18:44Z"
  name: sample-secret
  namespace: default
  resourceVersion: "50834"
  uid: 68d757b5-1623-4e58-ae03-81af0a8ec077
type: Opaque
root@minikube:~/tutorial#
```

`keyfile` と `message` が存在していることが確認できます。  
手動での生成を実施することができましたので、一旦作成したものを削除します:

```bash:minikube
root@minikube:~/tutorial# kubectl delete secret/sample-secret
secret "sample-secret" deleted
root@minikube:~/tutorial# kubectl get secret
No resources found in default namespace.
root@minikube:~/tutorial#
```

2.Secret と Pod を含むマニフェストファイルを作成しますが、Base64の文字列を取得しないといけないため、コマンドを実行します:

```bash:minikube
root@minikube:~/tutorial# echo -n 'Hello World !' | base64
SGVsbG8gV29ybGQgIQ==
root@minikube:~/tutorial#
```

```bash:minikube
root@minikube:~/tutorial# echo -n 'OUR-SECRET-KEY' | base64
T1VSLVNFQ1JFVC1LRVk=
root@minikube:~/tutorial#
```

それぞれ取得できましたので、これらを基にマニフェストファイルを作成します:

::: details secret.yml

```yaml:secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: sample-secret
data:
  message: SGVsbG8gV29ybGQgIQ==     # echo -n 'Hello World !' | base64
  keyfile: T1VSLVNFQ1JFVC1LRVk=     # cat ./keyfile | base64

---
apiVersion: v1
kind: Pod
metadata:
  name: sample
  namespace: default
spec:
  containers:
  - name: sample
    image: nginx:1.17.2-alpine
    env:
    - name: MESSAGE
      valueFrom:
        secretKeyRef:
          name: sample-secret
          key: message
    volumeMounts:
    - name: secret-storage
      mountPath: /home/nginx
  volumes:
  - name: secret-storage
    secret:
      secretName: sample-secret
      items:
      - key: keyfile
        path: keyfile
```

:::

3.Pod に入って Secret が接続されていることを確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f secret.yml
secret/sample-secret created
pod/sample created
root@minikube:~/tutorial#
```

```bash:minikube
root@minikube:~/tutorial# kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
sample   1/1     Running   0          36s
root@minikube:~/tutorial# kubectl exec -it sample -- sh
/ # ls /home/nginx/
keyfile
/ # cat /home/nginx/keyfile
YOUR-SECRET-KEY/ #
/ #
```

`keyfile` が接続されていることを確認できました。
演習としては完了なので、コンテナから抜けてPodを削除します。

---

### 永続データ (PersistentVolume, PersistentVolumeClaim)

#### PersistentVolume(PV) とは？

永続データの実態のこと。

- ストレージの接続情報
- ストレージの抽象化

**※下記はあくまで一例であり、調べてみた結果他にも種類がありました。**  
**その他の種類については公式ドキュメントを参照してください。**

https://kubernetes.io/docs/concepts/storage/persistent-volumes/

**▼マニフェストファイルの実装方法**:

```yaml:pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
  spec:
  # ストレージ抽象化を定義する3プロパティ (storageClassName, accessModes, capacity)
  storageClassName: host
  accessModes: [ReadWriteOnce]
  capacity:
  storage: 1Gi
  # 削除動作を定義するプロパティ (persistentVolumeReclaimPolicy)
  persistentVolumeReclaimPolicy: Retain
  # 保存先を定義するプロパティ (hostPath)
  hostPath:
  path: /data/storage
  type: Directory
```

**ストレージ抽象化を定義する3プロパティ**:

- `storageClassName`: ストレージの種類を指定
- `accessModes`: アクセスモードを指定
  - `ReadWriteOnce`: 読み書き可能
  - `ReadOnlyMany`: 読み込みのみ可能
  - `ReadWriteMany`: 読み書き可能
- `capacity`: ストレージの容量を指定

**削除動作を定義するプロパティ**:

- `persistentVolumeReclaimPolicy`: 削除時の動作を指定
- `Retain`: 削除時にストレージは削除されない
- `Delete`: 削除時にストレージは削除される
- `Recycle`: (**Kubernetes v1.15以降で非推奨**)削除時にストレージは削除される

**保存先を定義するプロパティ**:

- `hostPath`: ホストマシンのパスを指定
- `path`: ホストマシンのパスを指定
- `type`: ホストマシンのパスのタイプを指定
  - `DirectoryOrCreate`: ディレクトリが存在しない場合は作成
  - `Directory`: ディレクトリが存在しない場合はエラー
  - `FileOrCreate`: ファイルが存在しない場合は作成
  - `File`: ファイルが存在しない場合はエラー

#### PersistentVolumeClaim(PVC) とは？

永続データを要求するもの。

**▼マニフェストファイルの実装方法**:

```yaml:pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
  spec:
  # ストレージ抽象化を定義する3プロパティ (storageClassName, accessModes, resources)
  storageClassName: host
  accessModes: [ReadWriteOnce]
  resources:
  requests:
  storage: 1Gi
```

PVと違い、PVCは要求する動きになっているので `requests` を指定します。

#### 演習

```txt:お題目
1.PVとPVCを含むマニフェストファイルの作成
2.リソースの作成
```

1.PVとPVCを含むマニフェストファイルを作成します:

::: details storage.yml

```yaml:storage.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: volume-01
  labels:
    env: study
spec:
  storageClassName: slow
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/data/storage"
    type: Directory

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: volume-claim
  labels:
    env: study
spec:
  storageClassName: slow
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

:::

リソースを作成する前に、ホストマシンにディレクトリを作成します:

```bash:minikube
root@minikube:~/tutorial# mkdir -p /data/storage
root@minikube:~/tutorial#
root@minikube:~/tutorial# ls /data
storage
root@minikube:~/tutorial#
```

2.リソースを作成します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f storage.yml
persistentvolume/volume-01 created
persistentvolumeclaim/volume-claim created
root@minikube:~/tutorial# kubectl get pvc,pv
NAME                                 STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/volume-claim   Bound    volume-01   1Gi        RWO            slow           <unset>                 57s

NAME                         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/volume-01   1Gi        RWO            Retain           Bound    default/volume-claim   slow           <unset>                          57s
root@minikube:~/tutorial#
```

出力結果が見にくいですが、 `volume-01` という名前のPVが作成されており、**PV側にPVCがマウントされている**ことが確認できます。  
演習としては完了なので、コンテナから抜けてPodを削除します。

---

### StatefulSet

StatefulSetには、以下のような特徴があります。

- Podの集合
- Podをスケールする際の名前が一定規則で付けられる
  - 再起動した際も、同じ名前で起動される
- 各Podに対して一意なPVCが割り当てられ、Podが削除されてもPVCは削除されない

**マニフェストファイルの実装方法**:

```yaml:statefulset.yml
apiVersion: apps/v1
kind: StatefulSet
(省略)
spec:
  (省略)
  updateStrategy:
    type: RollingUpdate
  serviceName: nginx
  template:
  (省略)
  volumeClaimTemplates:
```

マニフェストファイル記載方法としては、 Deployment とほぼ同じですが、 `spec` の中に `volumeMounts` と `volumes` が追加されています。

違いがあるとすれば、以下が主な違いになります。

- `strategy` ではなく **`updateStrategy`** を使用する
- `serviceName` (**HeadlessService**) を指定する(後述で説明)
- `volumeClaimTemplates` のテンプレートを定義する

StatefulSetについての詳しい内容は、以下公式ドキュメントを参照してください:  
https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

#### 演習

```txt:お題目
1.StatefulSet・Serviceのマニフェストファイルの作成
2.デバック用PodからService経由でPodにアクセス
```

1.StatefulSet・Serviceのマニフェストファイルを作成します:

::: details tutorial/statefulset.yml

```yaml:tutorist/statefulset.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: volume-01
spec:
  storageClassName: standard
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/storage
    type: Directory

---
apiVersion: v1
kind: Service
metadata:
  name: sample-svc
spec:
  clusterIP: None
  selector:
    app: web
    env: study
  ports:
  - port: 80
    targetPort: 80

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
      env: study
  revisionHistoryLimit: 14
  serviceName: sample-svc
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: study
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2-alpine
        volumeMounts:
        - name: storage
          mountPath: /home/nginx
  volumeClaimTemplates:
  - metadata:
      name: storage
    spec:
      storageClassName: standard
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 1Gi
```

:::

`StatefulSet` の箇所を切り出して解説します。

**StatefulSet仕様設定**:

```yaml:StatefulSet仕様設定
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
      env: study
  revisionHistoryLimit: 14
  serviceName: sample-svc
```

- `spec.replicas`: 作成するPodの数を1つに指定
- `spec.selector.matchLabels`: 管理対象のPodを識別するためのラベル条件
- `spec.revisionHistoryLimit`: 保持するリビジョン履歴の上限数を14に設定
- `spec.serviceName`: StatefulSetに関連付けるHeadless Serviceの名前を指定

**ボリュームクレームテンプレート（StatefulSet固有機能）**:

```yaml:ボリュームクレームテンプレート（StatefulSet固有機能）
volumeClaimTemplates:
- metadata:
    name: storage
  spec:
    storageClassName: standard
    accessModes:
    - ReadWriteMany
    resources:
      requests:
        storage: 1Gi
```

- `volumeClaimTemplates`: StatefulSet固有の機能で、各Podに個別のPVCを自動作成
- `volumeClaimTemplates.metadata.name`: 作成されるPVCの名前テンプレート
- `volumeClaimTemplates.spec.storageClassName`: 使用するStorageClassを指定
- `volumeClaimTemplates.spec.accessModes`: ボリュームのアクセスモードを指定
- `volumeClaimTemplates.spec.resources.requests.storage`: 要求するストレージ容量を1Giに設定

これから起動させていくのですが、その前に `/data` 配下に `storage` ディレクトリがあるか確認します:

```bash:minikube
root@minikube:~/tutorial# ls /data/
storage
root@minikube:~/tutorial#
```

問題なさそうなので、実際にPodを起動してみます:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f statefulset.yml
persistentvolume/volume-01 configured
service/sample-svc created
statefulset.apps/nginx created
root@minikube:~/tutorial#
```

```bash:minikube
root@minikube:~/tutorial# kubectl get all
NAME          READY   STATUS    RESTARTS   AGE
pod/nginx-0   1/1     Pending   0          3m41s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   8d
service/sample-svc   ClusterIP   None         <none>        80/TCP    3m41s

NAME                     READY   AGE
statefulset.apps/nginx   1/1     3m41s
root@minikube:~/tutorial#
```

2.デバック用PodからService経由でPodにアクセスします:

```bash:minikube
root@minikube:~/tutorial# kubectl run debug --image=centos:7 -it --rm --restart=Never -- sh
If you don't see a command prompt, try pressing enter.
sh-4.2#
```

- `kubectl run`: Podを作成するコマンド
- `--image`: コンテナイメージを指定
- `-it`: コンテナに入る
- `--rm`: コンテナを削除
- `--restart=Never`: コンテナを削除しない
- `-- sh`: コンテナに入る

CentOS7をを起動させ、一時的にデバック用Podとして一時的に起動させました。  
**通常であればIPアドレスがわからないところを、Service経由でアクセスできるようになります。**

```bash:minikube
sh-4.2# curl http://sample-svc/
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

想定通り、nginxのページが表示されました。  
演習としては完了なので、後片付けをしていきます。

---

### Ingress

クラスター内のServiceに対する外部からのアクセス(主にHTTP)を管理するAPIオブジェクトです。  
Ingressは負荷分散、SSL終端、名前ベースの仮想ホスティングの機能を提供します。  
IngressはURLでサービスを切り替えることができます。

**マニフェストファイルの実装方法**:

```yaml:マニフェストファイルの実装方法
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

- `path`: パスを指定
- `backend`: 転送先のサービスを指定
- `service.name`: 転送先のサービス名を指定
- `service.port.number`: 転送先のサービスのポート番号を指定

#### 演習

```txt:お題目
1.Deployment, Service を作成する
2.Ingressを作成する
3.外部からアクセスして動作確認
```

1.Deployment, Service を作成します:

::: details tutorial/ingress.yml(Deployment, Serviceのみ)

```yaml:tutorial/ingress.yml
# service
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
    env: study
  ports:
  - port: 80
    targetPort: 80

---
# deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      env: study
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: study
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2-alpine

---

```

:::

2.Ingressを作成します:
::: details tutorial/ingress.yml(Ingress)

```yaml:tutorial/ingress.yml
# ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

:::

全体像としては以下の通りです:
::: details tutorial/ingress.yml(全体像)

```yaml:tutorial/ingress.yml
# service
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
    env: study
  ports:
  - port: 80
    targetPort: 80

---
# deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
      env: study
  template:
    metadata:
      name: nginx
      labels:
        app: web
        env: study
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2-alpine

---
# ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

:::

3.外部からアクセスして動作確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl apply -f ingress.yml
service/web-svc created
deployment.apps/nginx created
ingress.networking.k8s.io/frontend created
root@minikube:~/tutorial#
```

```bash:minikube
root@minikube:~/tutorial# kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-7f5db5465d-bfhj2   1/1     Running   0          40s
pod/nginx-7f5db5465d-wcvxm   1/1     Running   0          40s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   9d
service/web-svc      ClusterIP   10.98.28.152   <none>        80/TCP    40s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           40s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-7f5db5465d   2         2         2       40s
root@minikube:~/tutorial#
```

下記のコマンドで、Ingressの状態を確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl get ingress,service,deploy
NAME                                 CLASS   HOSTS   ADDRESS   PORTS   AGE
ingress.networking.k8s.io/frontend   nginx   *                 80      4m7s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   9d
service/web-svc      ClusterIP   10.98.28.152   <none>        80/TCP    4m7s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           4m7s
root@minikube:~/tutorial#
```

正常であればingresの行にIPアドレスが表示されるのですが、今回は表示されません。  
これは、minikubeのIngress機能が有効になっていないためです。  
minikubeのIngress機能を有効にするには、minikubeからexitで抜けて、以下のコマンドをホストから実行します:

```bash:terminal
minikube addons enable ingress
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
💡  アドオンを有効にした後、「minikube tunnel」を実行することで、ingress リソースが「127.0.0.1」で利用可能になります
    ▪ registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.5.3 イメージを使用しています
    ▪ registry.k8s.io/ingress-nginx/controller:v1.12.2 イメージを使用しています
    ▪ registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.5.3 イメージを使用しています
🔎  ingress アドオンを検証しています...
🌟  'ingress' アドオンが有効です
```

```bash:terminal
minikube addons list | grep ingress

| ingress                     | minikube | enabled ✅   | Kubernetes                     |
| ingress-dns                 | minikube | disabled     | minikube                       |
```

Ingressが有効化されました。

Ingress Controllerのログを確認します:

```bash:terminal
minikube tunnel

✅  トンネルが無事開始しました

📌  注意: トンネルにアクセスするにはこのプロセスが存続しなければならないため、このターミナルはクローズしないでください ...

❗  frontend service/ingress は次の公開用特権ポートを要求します:  [80 443]
🔑  sudo permission will be asked for it.
🏃  frontend サービス用のトンネルを起動しています。
Password:
🏃  frontend サービス用のトンネルを起動しています。
🎉  トンネルが正常に開始されました
```

```bash:terminal
kubectl get pods -n ingress-nginx

NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-cn22q       0/1     Completed   0          11m
ingress-nginx-admission-patch-f8qjg        0/1     Completed   1          11m
ingress-nginx-controller-67c5cb88f-nh4wp   1/1     Running     0          11m
```

再度minikube内部に接続して、Ingressの状態を確認します:

```bash:minikube
root@minikube:~/tutorial# kubectl get ingress,service,deploy
NAME                                 CLASS   HOSTS   ADDRESS        PORTS   AGE
ingress.networking.k8s.io/frontend   nginx   *       192.168.49.2   80      26m

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   9d
service/web-svc      ClusterIP   10.98.28.152   <none>        80/TCP    26m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           26m
root@minikube:~/tutorial#
```

Ingressに外部IPアドレスが表示されました！

ブラウザにて、IngressのIPアドレスにアクセスしてnginxのページが表示されたら、Ingressの設定は完了です。

**代替案**:  
もし `minikube tunnel` コマンドに時間がかかりすぎる場合は、以下の代替方法を試してください：

```bash:terminal
kubectl port-forward service/web-svc 8080:80
```

これにより、ローカルホストの8080ポートがIngressの80ポートにフォワードされます。

その後、ブラウザで：

```bash:terminal
http://localhost:8080
```

![代替案.png](/images/kubernetes-tutorial/代替案.png)

## 🗑️ 後片付け

今回は多めにリソースを作成しましたので、すべて削除していきます🔥:

**StatefulSetの削除**:

```bash:minikube
root@minikube:~/tutorial# kubectl delete -f statefulset.yml
persistentvolume/volume-01 deleted
service/sample-svc deleted
statefulset.apps/nginx deleted
root@minikube:~/tutorial#
```

**古いPVCの削除**:

```bash:minikube
root@minikube:~/tutorial# kubectl delete pvc volume-claim
persistentvolumeclaim "volume-claim" deleted
root@minikube:~/tutorial#
```

**最終確認**:

```bash:minikube
root@minikube:~/tutorial# kubectl get pv,pvc
No resources found
root@minikube:~/tutorial#
```

```bash:minikube(※ClusterIPが1つだけ残っているのは想定通り)
root@minikube:~/tutorial# kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   9d
root@minikube:~/tutorial#
```

**ホスト上のディレクトリも削除（オプション）**:

```bash:minikube
root@minikube:~/tutorial# sudo rm -rf /data/storage
root@minikube:~/tutorial# ls -la /data/
total 8
drwxr-xr-x 2 root root 4096 Jun  3 11:04 .
drwxr-xr-x 1 root root 4096 May 25 08:17 ..
root@minikube:~/tutorial#
```

## 🎉 まとめ

長い時間ご覧いただき、ありがとうございました。
この記事では、Kubernetesの基本概念から実際のハンズオンまでを通して学習しました。

Kubernetesは学習コストが高いツールですが、コンテナオーケストレーションにおいて非常に強力な機能を提供します。今回の基礎学習を土台に、さらに深い理解を目指していきたいと思います。

次回は、より実践的なリソースについて学習しアウトプットしていく予定です。
