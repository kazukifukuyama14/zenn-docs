---
title: "NATインスタンスからRedisへの接続手順まとめ"
emoji: "☁️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["EC2", "NATインスタンス", "Redis"]
published: true
---

業務にて、アプリ制作チームより以下依頼がありました。

:::message
検証用アプリをRedisに接続して疎通試験をしたいので対応してほしい
:::

私が所属しているインフラチームで対応する内容について、以下と認識しました。

- NATインスタンス作成
- ローカル環境からNATインスタンスを経由させ、Redisクラスターへ接続

:::message alert

- Redisの中身についてはインフラ側では何も設定しない(あくまでローカルから疎通出来たらOKレベル)
- IPアドレスの変更すると不都合だとのことで、ElasticIPで固定する
  :::

## 📝 前提条件

- RedisサーバーのプライベートIP：`192.168.64.228`
- Redisサーバーの設定（`redis.conf`）：

  ```conf
  bind 0.0.0.0
  protected-mode no
  ```

- NATインスタンスに redis-cli（v8.0.2）がインストール済み

## 👷 RedisサーバーのIPアドレス確認手順

1. Redisサーバーが稼働しているEC2にログイン
2. 以下のコマンドでIPアドレスを確認：

```bash
ip addr show
# または
hostname -I
```

3. 出力されたIPアドレスをメモ（例：192.168.64.228）

## ⚡ Redisへの接続手順

```bash
# NATインスタンスにログイン
ssh -i <鍵ファイル.pem> ec2-user@<NATインスタンスのパブリックIP>

# redis-cli のあるディレクトリに移動
cd /root/redis-stable

# Redisに接続
./src/redis-cli -h 192.168.64.228 -p 6379

# 接続確認
ping
# → PONG が返れば成功
```

## 🌀 トラブルシューティング

| チェック項目          | 内容                                                            |
| --------------------- | --------------------------------------------------------------- |
| Redisが起動しているか | `ps aux`                                                        |
| bind設定              | bind 0.0.0.0 になっているか                                     |
| protected-mode        | no になっているか                                               |
| セキュリティグループ  | NATインスタンス → Redisサーバーへの TCP:6379 が許可されているか |

## ✏ 接続確認で躓いたポイントまとめ

接続確認で躓いたポイントまとめ:

1. 誤ったIPアドレスの指定

- 192.168.16.0 はネットワークアドレスであり、ホストには使用不可。
- ip addr show や hostname -I で正しいIPを確認。

2. Redisの設定が外部接続を拒否

- bind が 127.0.0.1、protected-mode が yes だった。
- 対処：bind 0.0.0.0、protected-mode no に変更 → Redis再起動。

3. redis-server バイナリが存在しない

- make 実行前に redis-server が未生成。
- 対処：make を実行してビルド。

4. Redis未起動で接続試行

- Connection refused エラー。
- 対処：./src/redis-server redis.conf で起動。

5. クラスター構成と誤認

- cluster nodes 実行でエラー。
- 対処：redis.conf の cluster-enabled を確認 → 単一ノード構成。

### EC2インスタンス構成

- ElasticIPでIP固定
- インスタンスタイプ：t2.micro
- サブネット：パブリック
- AMI：Amazon Linux 2023
- pemキー発行済み
- NAT設定あり

### セキュリティグループ（SG）構成

- NATインスタンス → Redisサーバーへの TCP:6379 を許可
- Redis ClusterはAurora用SGを使用
- Redis用サブネットグループを作成（2つあるが1つのみ使用）

### VPC関連構成

- 検証用VPCを使用
- サブネット：
- パブリック：NATインスタンス用、インターネットゲートウェイ設定済み
- プライベート：0.0.0.0/0 をルートテーブルに設定
- ルートテーブル：
- パブリック：0.0.0.0/0（IGW）、192.168.0.0/16（VPC）
- プライベート：0.0.0.0/0（NAT経由）

## 💻 NATインスタンスに redis-cli をインストールする手順

### 1. 基本確認

- OS：Amazon Linux 2023
- インターネットアクセス可能
- パッケージ管理ツール：dnf

### 2. セキュリティグループとIAMロール

- SG：SSH（22番）など必要なポートが開いているか
- IAMロール：必要に応じてS3などのアクセス権を付与

### 3. ビルドツールと依存パッケージのインストール

```bash
sudo dnf groupinstall "Development Tools" -y
sudo dnf install gcc make wget tar -y
```

### 4. Redis ソースコードのダウンロードとビルド

```bash
wget http://download.redis.io/redis-stable.tar.gz
tar -xvzf redis-stable.tar.gz
cd redis-stable
make
```

### 5. redis-cli の配置

```bash
sudo cp src/redis-cli /usr/local/bin/
```

### 6. 動作確認

```bash
redis-cli --version
```

## 参考リンク

### ☁ AWS関連

https://qiita.com/kamome_ss/items/b3e82cc6e3cbf812a233  
https://qiita.com/seikida/items/8a68ceabfd1342f636bd  
https://dev.classmethod.jp/articles/how-to-create-amazon-linux-2023-nat-instance/#%25E3%2580%25902023%25E5%25B9%25B411%25E6%259C%258813%25E6%2597%25A5-%25E8%25BF%25BD%25E8%25A8%2598%25E3%2580%2591aws%25E5%2585%25AC%25E5%25BC%258F%25E3%2583%2589%25E3%2582%25AD%25E3%2583%25A5%25E3%2583%25A1%25E3%2583%25B3%25E3%2583%2588%25E3%2581%25AE%25E6%2589%258B%25E9%25A0%2586  
https://dev.classmethod.jp/articles/nat-instance-handmaid/#3.%2520%25E3%2582%25AF%25E3%2583%25A9%25E3%2582%25A4%25E3%2582%25A2%25E3%2583%25B3%25E3%2583%2588%25E3%2582%25A4%25E3%2583%25B3%25E3%2582%25B9%25E3%2582%25BF%25E3%2583%25B3%25E3%2582%25B9%25E3%2581%25AE%25E3%2582%25B5%25E3%2583%2596%25E3%2583%258D%25E3%2583%2583%25E3%2583%2588%25E3%2583%25AB%25E3%2583%25BC%25E3%2583%2588%25E3%2583%2586%25E3%2583%25BC%25E3%2583%2596%25E3%2583%25AB%25E3%2581%25AE%25E5%25A4%2589%25E6%259B%25B4

### 🔒 ネットワーク・ファイアウォール設定（Linux）

https://atmarkit.itmedia.co.jp/ait/articles/1709/22/news019.html  
https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/9/html/configuring_firewalls_and_packet_filters/configuring-nat-using-nftables_getting-started-with-nftables  
https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_firewalls_and_packet_filters/getting-started-with-nftables_firewall-packet-filters
