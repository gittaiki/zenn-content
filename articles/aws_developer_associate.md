---
title: "【AWS DVA】受験動機と合格するためにしたこと"
emoji: "🗂"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [aws認定試験, aws, 資格]
published: true
---

## はじめに

先日AWS 認定デベロッパー - アソシエイト(DVA-C02)試験を受験し、一発合格しました。
DVAを受験しようと思った動機と、合格するためにしたことを備忘録として残していきます。
:::message
ただの備忘録です。参考にするかは自己責任でお願いします。
:::

https://twitter.com/_taaaaiki/status/1751814276279644480

## 使ったことがあるAWSサービス

仕事や個人開発で使ったことがあるAWSサービスは以下です。
- 仕事
  - API Gateway, Lambda, DynamoDB, CloudFormationなど

  SAM(Serverless Application Model)を使用してサーバーレスアプリケーションを開発した経験があります。この経験が受験動機に繋がります。


- 個人開発
  - ECR, ECS, EC2, RDS, S3, ElastiCache, CloudWatchなど

  ECSとEC2それぞれにRailsアプリケーションをデプロイした経験があります。


## 受験動機

仕事でサーバーレスアプリケーションの開発に携わるまで、「SAM？何それ？」って状態でした。そのためキャッチアップから入り、何とか環境構築と開発、そしてデプロイすることはできました。
無事実装はできたものの「シンプルに実装できたのか？」や、「IAMポリシーを最低限にするまでに時間掛かったなー」など色々モヤモヤが残っていました。
このモヤモヤを解消するため、AWS を使用したアプリケーションの開発、デプロイについて一通り学習したいと思いました。この学習したい内容がDVA-C02の試験内容と近かったため、受験することを決めました。

## 合格するためにしたこと

### ポケットスタディ[DVA-C01版]

2週しました。
わからなかった用語（例：スロットリング, ポーリングなど）はNotionにまとめていました。
マネジメントコンソール上での表示画面や、図が多くわかりやすかったです。
（[DVA-C01版]購入して1週間後、[DVA-C02対応版]が発売していました😂👇）
https://www.amazon.co.jp/%E3%83%9D%E3%82%B1%E3%83%83%E3%83%88%E3%82%B9%E3%82%BF%E3%83%87%E3%82%A3-AWS%E8%AA%8D%E5%AE%9A%E3%83%87%E3%83%99%E3%83%AD%E3%83%83%E3%83%91%E3%83%BC%E3%82%A2%E3%82%BD%E3%82%B7%E3%82%A8%E3%82%A4%E3%83%88-%EF%BC%BBDVA-C02%E5%AF%BE%E5%BF%9C%EF%BC%BD-%E5%B1%B1%E4%B8%8B%E5%85%89%E6%B4%8B/dp/4798070696/ref=sr_1_1?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=M9OCSX0K6N3M&keywords=%E3%83%9D%E3%82%B1%E3%83%83%E3%83%88%E3%82%B9%E3%82%BF%E3%83%87%E3%82%A3+dva&qid=1706929810&sprefix=%E3%83%9D%E3%82%B1%E3%83%83%E3%83%88%E3%82%B9%E3%82%BF%E3%83%87%E3%82%A3+dva%2Caps%2C164&sr=8-1

### 模擬試験

ポケットスタディに想定問題が50問ありますが、それだけでは心許なかったため以下2つの模擬試験をしました。
1. AWS Skill Builder
`AWS 認定デベロッパー - アソシエイト(DVA-C02 - Japanese)`を3週しました。
1問解答するごとに正解と解説が表示されるため、すぐ復習ができます。

https://explore.skillbuilder.aws/learn/course/14060/aws-certified-developer-associate-official-practice-question-set-dva-c02-japanese

2. Udemy
`【DVA-C02】AWS Certified Developer - Associate | 模擬試験`を3週しました。
各セクションごとの正解率が表示されるため、自分の苦手領域がわかります。
本番の試験では3問くらい同じ問題が出題されました。

https://www.udemy.com/course/dva-c02aws-certified-developer-associate-p/

### **AWS ハンズオン資料**

仕事でサーバーレスアプリケーションの開発に携わる際、キャッチアップで利用しました。
DVA取得のためだけなら必要ないかもですが、DVAの学習効率を上げたい方や、サーバーレスアプリケーションの開発をする予定がある方は、以下2つのハンズオンをやって損はないと思います。
https://pages.awscloud.com/JAPAN-event-OE-Hands-on-for-Beginners-Serverless-1-2022-reg-event.html?trk=aws_introduction_page

https://pages.awscloud.com/JAPAN-event-OE-Hands-on-for-Beginners-Serverless-2-2022-reg-event.html?trk=aws_introduction_page

## 受験後の感想

まず合格できてほっとしました😮‍💨
あと受験動機で話したモヤモヤは解消しました🎉
改めて実装したサーバーレスアプリケーションを見てみると、「シンプルに実装できていない（よくない）」と自分で判断できます。例えば、環境ごとにテンプレートファイルやLambdaを作ってしまっている点です（反省）。
この反省を踏まえて、今後リファクタリングや、新しくサーバーレスアプリケーションを実装する際に活かしたいと思います！
