---
title: "【SAM】Lambda + API Gatewayの開発環境構築"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, sam, lambda, apigateway, python]
published: true
---

## はじめに

SAMを使って、ローカルでAPI Gatewayを起動し、lambda_function.pyにリクエストできる環境を構築します

## SAMとは

AWSでサーバーレスアプリケーションを構築するためのフレームワークです。
このフレームワークを使用することで、AWS Lambda、Amazon API Gateway、およびその他のサーバーレスリソースのアプリケーションを定義、デプロイすることができます。
SAM CLIを使用すると、ローカルの開発環境でAWS LambdaやAmazon API Gatewayなどを`模倣`することができます。

## 前提

以下の内容は、インストール済みとします。
- AWS CLI
- Docker

AWSのアクセスキーとシークレットアクセスキーは`default`で設定済みとします。

```bash
# ~/.aws/credentials
[default]
aws_access_key_id = ***
aws_secret_access_key = ***
```

## SAMをインストール

HomebrewでSAM CLIをインストールします

```bash
# aws-sam-cliをインストールできるようにする
$ brew tap aws/tap
# インストール
$ brew install aws-sam-cli
# バージョンが出力されていればインストール成功
$ sam --version
```

## ディレクトリ構成

以下のディレクトリ構成になるようlambda_function.pyを作成します

```bash
.
`-- lambda_function.py
```

## 構成を定義

ローカルの開発環境でAWSのLambdaと、API Gatewayを模倣するための構成をYAML形式で定義します

- template.yamlの作成

```bash
$ cd .
$ touch template.yaml
```

- template.yamlに構成を定義

```yaml
# template.yaml
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
      StageName: development
      EndpointConfiguration: REGIONAL
```

定義の内容は、公式のリファレンスが参考になります👇
https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-resources-and-properties.html

## API Gatewayをローカルで実行

```bash
# ビルド
$ sam build
# 実行
$ sam local start-api
```

以上でAPI Gatewayがローカルで起動しています。
lambda_function.pyが呼び出せているか確認のため、`print`で標準出力させます。

```bash
# lambda_function.py
def lambda_handler(event, context):
    print('Hello, World!') # 追加
```

[http://127.0.0.1:3000/api](http://127.0.0.1:3000/api) にアクセスして、以下のログが出力されていると開発の環境構築は成功です🎉

```bash
Invoking lambda_function.lambda_handler (python3.9)

Hello, World!
```

## 参考記事

https://pages.awscloud.com/JAPAN-event-OE-Hands-on-for-Beginners-Serverless-2-2022-confirmation_009.html