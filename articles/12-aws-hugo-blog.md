---
title: "AWS + Hugo で技術ブログを構築しました"
emoji: "📖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS, Terraform, Hugo, GitHub Actions, ブログ]
published: true
---

## はじめに

今後は自分のブログにアウトプットして育てていきたいと思います。

https://wan0ri.com/

## 概要

- AWS と Hugo の組み合わせで技術ブログを構築した。
- `staging` : リポジトリ公開用。stagingはローカル検証用で構築したので **ACM** と **Route53** の構築は省略。
- `prod` : リポジトリはPrivateで管理。

https://github.com/kazukifukuyama14/techblog-staging

## コンセプト条件の整理

| 条件                                  | 技術選定の方向性                          |
| ------------------------------------- | ----------------------------------------- |
| コストを抑える                        | 無料枠や低料金サービスを活用              |
| フロント/バックエンド知識が浅い       | GUIや簡単なCLIで操作可能なツールを選定    |
| インフラはTerraformで管理             | IaC（Infrastructure as Code）を前提に構成 |
| ドメインと証明書はRoute 53とACMで管理 | AWSサービスで統一管理                     |

## 推奨技術スタック一覧

| 機能                      | 推奨技術                      | 理由                                     |
| ------------------------- | ----------------------------- | ---------------------------------------- |
| 静的サイト生成            | Hugo                          | 高速・簡単・Markdownベース               |
| ストレージ & ホスティング | Amazon S3                     | 静的サイトホスティングに最適、無料枠あり |
| CDN配信                   | Amazon CloudFront             | 高速配信、ACMと連携可能                  |
| ドメイン管理              | Amazon Route 53               | DNS管理、Terraform対応                   |
| SSL証明書                 | AWS Certificate Manager (ACM) | 無料、CloudFrontと連携可能               |
| IaC管理                   | Terraform                     | AWS全体をコードで管理可能                |
| CI/CD                     | GitHub Actions                | Hugoビルド → S3アップロードを自動化可能  |

## プロジェクトディレクトリ構成（詳細版）

本番環境では以下構成で構築しました：

```bash
aws-hugo-blog/
├── hugo-site/                       # Hugoプロジェクト（静的サイト生成）
│   ├── config.toml                  # Hugo設定ファイル
│   ├── content/                     # Markdown記事
│   ├── layouts/                     # テンプレート
│   ├── static/                      # 静的ファイル（画像・CSSなど）
│   └── themes/                      # Hugoテーマ
│
├── terraform/                       # TerraformによるAWSインフラ管理
│   ├── main.tf                      # メイン設定ファイル（モジュール呼び出し）
│   ├── variables.tf                 # 変数定義
│   ├── outputs.tf                   # 出力定義
│   ├── terraform.tfvars             # 変数値（環境ごとに分けてもOK）
│   ├── provider.tf                  # AWSプロバイダー設定
│   ├── modules/                     # モジュール化された構成
│   │   ├── s3/                      # S3バケット構成
│   │   ├── cloudfront/              # CloudFront構成
│   │   ├── route53/                 # Route 53ゾーン & レコード
│   │   └── acm/                     # SSL証明書（ACM）
│   │
├── .github/
│   └── workflows/
│       └── deploy.yml               # GitHub ActionsによるCI/CD（Hugo → S3）
│
├── README.md                        # プロジェクト概要・構築手順
└── LICENSE                          # ライセンス（任意）
```

### 各ディレクトリの役割

- hugo-site/：Hugoで生成する静的サイトのソース。Markdownで記事を書き、テーマでデザインを整える。
- terraform/：AWSリソース（S3, CloudFront, Route 53, ACM）をコードで管理。
- modules/：Terraformの構成をモジュール化することで再利用性と可読性を向上。
- .github/workflows/：GitHub ActionsでCI/CDを構成。記事更新 → 自動ビルド → S3へアップロード。

## AWSリソース一覧（Terraform管理対象）

### 1. 静的サイトホスティング関連

| リソース名                       | Terraformリソースタイプ               | 用途                                     |
| -------------------------------- | ------------------------------------- | ---------------------------------------- |
| S3バケット                       | aws_s3_bucket                         | Hugoで生成した静的ファイルのホスティング |
| S3バケットポリシー               | aws_s3_bucket_policy                  | 公開アクセス設定（静的サイト用）         |
| CloudFrontディストリビューション | aws_cloudfront_distribution           | CDN配信、SSL証明書との連携               |
| CloudFrontオリジンアクセスID     | aws_cloudfront_origin_access_identity | S3への安全なアクセス                     |

### 2. ドメイン & DNS関連

| リソース名                   | Terraformリソースタイプ              | 用途                                                              |
| ---------------------------- | ------------------------------------ | ----------------------------------------------------------------- |
| Route 53ホストゾーン         | aws_route53_zone                     | ドメインのDNS管理                                                 |
| DNSレコード                  | aws_route53_record                   | A/AAAA/CNAMEなどのレコード設定                                    |
| ドメイン登録情報（読み取り） | aws_route53domains_registered_domain | AWSで取得したドメインの登録情報を管理（※新規登録はTerraform不可） |

### 3. SSL/TLS証明書関連

| リソース名      | Terraformリソースタイプ | 用途                            |
| --------------- | ----------------------- | ------------------------------- |
| ACM証明書       | aws_acm_certificate     | CloudFront用のSSL証明書（無料） |
| DNS検証レコード | aws_route53_record      | ACM証明書のDNS検証用レコード    |

### 4. IAM関連

| リソース名                | Terraformリソースタイプ        | 用途                                        |
| ------------------------- | ------------------------------ | ------------------------------------------- |
| IAMユーザー               | aws_iam_user                   | TerraformやCI/CD用のアクセスユーザー        |
| IAMポリシー               | aws_iam_policy                 | S3/CloudFrontなどへのアクセス権限           |
| IAMポリシーアタッチメント | aws_iam_user_policy_attachment | ユーザーにポリシーを紐付け                  |
| IAMアクセスキー           | aws_iam_access_key             | TerraformやGitHub Actionsで使用する認証情報 |

### 5. オプション（CI/CDやログ管理など）

| リソース名                | Terraformリソースタイプ  | 用途                                        |
| ------------------------- | ------------------------ | ------------------------------------------- |
| CloudWatchロググループ    | aws_cloudwatch_log_group | CloudFrontやS3のログ管理                    |
| GitHub Actions用IAMロール | aws_iam_role             | OIDC連携によるセキュアなCI/CD（高度な構成） |

## GitHub Actionsで自動化

### .github/workflows/deploy.yml（構成例）

```yaml
name: Deploy Hugo Blog to AWS

on:
  push:
    branches:
      - main # メインブランチにpushされたときに実行

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    env:
      AWS_REGION: ap-northeast-1
      S3_BUCKET: your-s3-bucket-name
      CLOUDFRONT_DISTRIBUTION_ID: your-distribution-id

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "0.121.1" # 使用するHugoのバージョン

      - name: Build Hugo site
        run: hugo --minify

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Deploy to S3
        run: |
          aws s3 sync public/ s3://${{ env.S3_BUCKET }} \
            --delete \
            --acl public-read \
            --cache-control "max-age=0, no-cache, no-store, must-revalidate"

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ env.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

### この構成のポイント

- Hugoのビルドは hugo --minify で圧縮付きで生成。
- AWS認証情報は GitHub Secrets に登録して安全に管理。
- S3へのアップロードは aws s3 sync で差分更新。
- CloudFrontキャッシュの無効化で即時反映。

### GitHub Secrets に登録すべき項目

| 名前                  | 内容                          |
| --------------------- | ----------------------------- |
| AWS_ACCESS_KEY_ID     | IAMユーザーのアクセスキー     |
| AWS_SECRET_ACCESS_KEY | IAMユーザーのシークレットキー |
