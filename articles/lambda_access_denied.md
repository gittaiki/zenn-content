---
title: "【AWS SAM】Lambda同士の呼び出しで「AccessDeniedException」になる理由と対処法"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, sam, lambda, permission, iam]
published: true
---

## はじめに

AWS SAMでLambda A（リクエスト受付）から Lambda B（処理実行）を呼び出す構成を組みました。

```
RequestFunction（Lambda A） → ProcessorFunction（Lambda B）
```

呼び出す側のLambda Aに `lambda:InvokeFunction` 権限を付与していたので問題ないと思っていたが、Lambda B（処理実行）へのアクセス権限エラーが発生しました。
今回はこのエラー原因と解決をします。

## 構成

`template.yaml`の内容を一部抜粋

```yaml
Resources:
  # Lambda A
  RequestFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: request.app.lambda_handler
      Runtime: python3.9
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /process
            Method: post
      Policies:
        - Version: "2012-10-17"
          Statement:
            - Effect: Allow
              Action:
                - lambda:InvokeFunction
              Resource: !GetAtt ProcessorFunction.Arn
  # Lambda B
  ProcessorFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: processor.app.lambda_handler
      Runtime: python3.9
```

RequestFunctionのPoliciesに`lambda:InvokeFunction`を書いたため、呼び出せると思っていました。
しかし、デプロイ後にRequestFunctionを実行すると、以下のようなエラーが発生しました。
```
[ERROR] ClientError: An error occurred (AccessDeniedException) when calling the Invoke operation: User: arn:aws:sts::*** is not authorized to perform: lambda:InvokeFunction on resource: arn:aws:lambda:*** because no identity-based policy allows the lambda:InvokeFunction action
```

## 原因

呼び出し**される側**の権限が許可されていなかった。

| 権限の種類 | 対象 | 役割 |
| --- | --- | --- |
| IAM Policy | 呼び出す側のLambda A | `lambda:InvokeFunction`を使うための許可 |
| Lambda Permission | 呼び出される側のLambda B | 「誰からの呼び出しを受け入れるか」を明示する |

つまり、Policiesに`lambda:InvokeFunction`を書くだけでは不十分で、呼び出されるProcessorFunctionに`Lambda::Permission`を追加する必要がありました。

## 解決方法

`Lambda::Permission`を`template.yaml`に追加したところ、無事に呼び出しできるようになりました🎉

```yaml
Resources:
  ProcessorFunctionPermission:
    Type: AWS::Lambda::Permission
    Properties:
      Action: lambda:InvokeFunction
      FunctionName: !GetAtt ProcessorFunction.Arn
      Principal: lambda.amazonaws.com
      SourceArn: !GetAtt RequestFunction.Arn
```

項目の補足：

- `Action`: Lambdaを呼び出す操作を許可
- `FunctionName`: 呼び出される関数（ProcessorFunction）
- `SourceArn`: 呼び出し元のLambda関数（RequestFunction）

## おわりに

DynamoDBやS3にアクセスする場合は、IAMロールに必要な権限を付与すればアクセスできます。
しかし、Lambdaを呼び出す場合はそれだけでは不十分で、呼び出される側の関数に対しても、誰からの呼び出しを許可するかを設定する必要があることがわかりました。

## 参考記事

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-permission.html
