---
title: "【GitHub Actions】Lambda + API Gatewayの自動デプロイを実装"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [githubactions, aws, lambda, apigateway, cicd]
published: true
---

## はじめに

以前作成した記事[【SAM】Lambda + API Gatewayの開発環境構築](https://zenn.dev/y_taiki/articles/aws_sam_setup)では開発環境を構築しました。
今回は本番環境を構築し、AWS上にLambdaとAPI Gatewayを、GitHub Actionsで自動デプロイするように実装します。

## 前提

以下条件の`IAMロール`は、AWSで作成済みとします。
ウェブアイデンティティプロバイダー
- token.actions.githubusercontent.com

ポリシーのサービス
1. S3
2. CloudFormation
3. Lambda
4. API Gateway

:::message alert
ポリシーのアクション・リソース等は適宜設定してください
:::

## ディレクトリ構成

```bash
.
|-- lambda_function.py
|-- requirements.txt
`-- template.production.yaml
```

template.production.yamlファイルに本番環境の定義を記述します。

```yaml:template.production.yaml
Transform: AWS::Serverless-2016-10-31
Resources:
  SampleLambda: # Lambda
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: sample_lambda
      CodeUri: ./
      Handler: lambda_function.lambda_handler
      Runtime: python3.9
      Timeout: 30
      MemorySize: 256
      Events:
        GetApi:
          Type: Api
          Properties:
            Path: /api
            Method: GET
            RestApiId:
              Ref: SampleAPI

  SampleAPI: # API Gateway
    Type: AWS::Serverless::Api
    Properties:
      Name: sample_api
      StageName: production
      EndpointConfiguration: REGIONAL
```

## 自動デプロイを実装

### 作業ブランチを作成&移動

```bash
$ git checkout -b github_actions
```

### S3作成

デプロイ時に、コードとリソース等をアップロードするためのS3を作成します。

```bash
aws s3 mb s3://sample_bucket
```

:::message
AWS CLIをインストールしていない場合、AWSマネジメントコンソールから作成してください
:::

### ワークフロー作成

GitHub Actionsのワークフローファイルを作成します。

```bash
$ mkdir -p .github/workflows
$ touch .github/workflows/deploy.yml
```

deploy.ymlファイルには、AWS上へ自動デプロイするための定義を記述します。

```yml:deploy.yml
name: production

on: # 自動デプロイ確認のため、作業ブランチを指定しています
  push:
    branches:
      - github_actions

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup python
        uses: actions/setup-python@v3
        with:
          python-version: '3.9'

      - name: Setup aws-sam
        uses: aws-actions/setup-sam@v2

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: ap-northeast-1
          role-to-assume: ${{ secrets.IAM_ROLE_ARN }}

      - name: Build
        run: sam build --template-file template.production.yaml

      - name: Deploy
        run: |
          sam deploy \
              --template-file .aws-sam/build/template.yaml \
              --stack-name ${{ secrets.CFN_STACK_NAME }} \
              --s3-bucket ${{ secrets.S3_BUCKET_NAME }} \
              --no-fail-on-empty-changeset
```

### GitHubに環境変数設定

リポジトリの`Settings > Secrets and variables > Actions`より、GitHub Actionsで使用する環境変数を設定します。

![](/images/sam_deploy_with_githubactions/01_image.png)

`New repository secret`ボタンを押下し、deploy.ymlファイルで使用している環境変数を以下のように設定します。

![](/images/sam_deploy_with_githubactions/02_image.png)

以上で実装終了です。
`github_actions`ブランチの変更内容をGitHubにpushすると、GitHub Actionsが実行されます。
ワークフローの実行内容は`Actions`から確認できます。
Statusが`Success`✅になると、自動デプロイ完了です🎉
AWSマネジメントコンソールではLambdaと、API Gatewayが正常にデプロイされていることを確認できます。

## 参考記事

https://github.com/aws-actions/setup-sam
