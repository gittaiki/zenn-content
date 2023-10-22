---
title: "【SAM】Unable to import module を解決"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, sam, awssam, lambda, python]
published: true
---

## はじめに

以前作成した記事[【SAM】Lambda + API Gatewayの開発環境構築](https://zenn.dev/y_taiki/articles/aws_sam_setup)で構築した開発環境を、AWS上にデプロイしました。しかし`requirements.txt`に書いているモジュールがインポートできず、Lambdaでエラーが発生していました。
今回はそのエラーを解決します。

## ディレクトリ構成

```bash
.
|-- lambda_function.py
|-- requirements.txt
`-- template.yaml
```

各ファイルの内容は以下です👇

```txt:requirements.txt
slackweb
```

```yaml:template.yaml
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

## 実行したコマンド

以下の方法でデプロイすると、`template.yaml`に定義したLambdaとAPI GatewayがAWS上に作成されました。

```bash
$ sam build
$ sam deploy --stack-name test_lambda --s3-bucket test_lambda
```

しかしLambdaが実行する度、`requirements.txt`に書いたモジュール`slackweb`がありませんとエラーが発生します。

```bash
[ERROR] Runtime.ImportModuleError: Unable to import module 'lambda_function': No module named 'slackweb'
Traceback (most recent call last):
```

## 解決方法

ビルド後、`.aws-sam/build`配下に`template.yaml`が生成されます。
その生成された`template.yaml`を指定してデプロイすると、ビルド時にインストールしたモジュールがインポートできるようになり、解決しました🎉

```bash
$ sam build
$ sam deploy --template-file .aws-sam/build/template.yaml --stack-name test_lambda --s3-bucket test_lambda
```

## 参考記事

https://dev.classmethod.jp/articles/aws-sam-simplifies-deployment/
