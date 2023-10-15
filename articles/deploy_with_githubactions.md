---
title: "【Github Actions】ECSサービスを更新してデプロイ"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [githubactions, aws, ecs, ecr, cicd]
published: true
---

## はじめに

[Github Docs](https://docs.github.com/ja/actions/deployment/deploying-to-your-cloud-provider/deploying-to-amazon-elastic-container-service)を参考にすると、ECSに自動デプロイするGithub Actionsが作成できます。Github Actionsが実行される度、ECSのタスク定義に新しいリビジョンのタスクが新規作成され、デプロイします。
今回GIthub Actionsが実行されると、新しいリビジョンのタスクを新規作成されず、実行中のタスクを新鮮な状態にして、デプロイするようにしたいと思います。

## 用意されたワークフローを見てみる

2023/10/15現在Github Docsの[ワークフローの作成](https://docs.github.com/ja/actions/deployment/deploying-to-your-cloud-provider/deploying-to-amazon-elastic-container-service#creating-the-workflow)での定義は以下になります。
今回`docker push`コマンドでイメージをECRにpushした後の処理は削除します。

```yaml
name: Deploy to Amazon ECS

on:
  push:
    branches:
      - main

env:
  AWS_REGION: MY_AWS_REGION                   # set this to your preferred AWS region, e.g. us-west-1
  ECR_REPOSITORY: MY_ECR_REPOSITORY           # set this to your Amazon ECR repository name
  ECS_SERVICE: MY_ECS_SERVICE                 # set this to your Amazon ECS service name
  ECS_CLUSTER: MY_ECS_CLUSTER                 # set this to your Amazon ECS cluster name
  ECS_TASK_DEFINITION: MY_ECS_TASK_DEFINITION # set this to the path to your Amazon ECS task definition
                                               # file, e.g. .aws/task-definition.json
  CONTAINER_NAME: MY_CONTAINER_NAME           # set this to the name of the container in the
                                               # containerDefinitions section of your task definition

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@0e613a0980cbf65ed5b322eb7a1e075d28913a83
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@62f4f872db3836360b72999f4b87f1ff13310f3a

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Build a docker container and
          # push it to ECR so that it can
          # be deployed to ECS.
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

# 以降の処理はすべて削除
      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@c804dfbdd57f713b6c079302a4c01db7017a36fc
        with:
          task-definition: ${{ env.ECS_TASK_DEFINITION }}
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}

      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@df9643053eda01f169e64a0e60233aacca83799a
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

## 定義を修正

イメージをECRにpushした後、`aws ecs update-service` コマンドでECSのサービスを更新します。さらに`--force-new-deployment` オプションを付け、デプロイを開始するようにします。

```yaml
# 以上省略
env:
  AWS_REGION: MY_AWS_REGION                   # set this to your preferred AWS region, e.g. us-west-1
  ECR_REPOSITORY: MY_ECR_REPOSITORY           # set this to your Amazon ECR repository name
  ECS_SERVICE: MY_ECS_SERVICE                 # set this to your Amazon ECS service name
  ECS_CLUSTER: MY_ECS_CLUSTER                 # set this to your Amazon ECS cluster name
  ECS_TASK_DEFINITION: MY_ECS_TASK_DEFINITION # set this to the path to your Amazon ECS task definition
                                               # file, e.g. .aws/task-definition.json
  CONTAINER_NAME: MY_CONTAINER_NAME           # set this to the name of the container in the
                                               # containerDefinitions section of your task definition

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
      # ECRにイメージをpushする処理まで省略

# 以下の処理を追加
      - name: Update service
        run: |
          aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment
```

以上で変更は終了です。
Github Actionsが実行されると、新しいリビジョンのタスクが新規作成されず、実行中のタスクを新鮮な状態にしてデプロイすることができました🎉

## 最後に

デプロイされる度にタスクが新規作成されるのを避けたい場合、是非試してみてください。

## 参考記事

https://docs.github.com/ja/actions/deployment/deploying-to-your-cloud-provider/deploying-to-amazon-elastic-container-service#creating-the-workflow

https://docs.aws.amazon.com/cli/latest/reference/ecs/update-service.html
