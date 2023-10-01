---
title: "【簡単】ngrokで発行されるURLを固定する"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ngrok]
published: true
---

## はじめに

LINEチャットボットを開発するためには、HTTPSのURLが必要です。そのため一時的にHTTPSのURLを生成できるngrokを使用しています。ngrokで生成されるURLのドメインは毎回変更されるため、その度URLをLINEに設定していました。
今回、発行されるURLのドメインを固定する設定を行います。

## 事前準備

### アカウント作成👇

https://dashboard.ngrok.com/login

### ngrokをインストール

https://ngrok.com/docs/getting-started/#step-2-install-the-ngrok-agent

例）MacOSでHomeBrewを使用している場合

```bash
$ brew install ngrok/ngrok/ngrok
```

## 設定

### ngrokを認証

サイドバーGetting Started > Your Authtokenより、認証トークンをコピー

![](/images/ngrok_domain/01_image.png)

`ngrok config` コマンドでコピーした認証トークンを設定
※これを行う必要があるのは初回だけです

```bash
$ ngrok config add-authtoken <認証トークン>
```

### 独自ドメインを作成

サイドバーCloud Edge > Domainsより、「+ Create Domain」ボタンで生成されたドメインをコピー

![](/images/ngrok_domain/02_image.png)

### 独自ドメインで一時的公開

`ngrok http` コマンドで、ローカルで実行しているHTTPサービスをHTTPSで公開できます。さらに`--domain` オプションで、URL(`https://<コピーしたドメイン>`)を生成することができます。

```bash
$ ngrok http --domain=<コピーしたドメイン> <ポート（例:3000）>
```

以上で、固定されたドメインでHTTPSのURLの生成は完了🎉
2回目以降も上のコマンドでHTTPサービスを公開しても、ドメインが固定されています。LINEチャットボット等の開発の際は是非試してみてください。

## 参考記事

https://zenn.dev/u2dncx/articles/3fa29c8a3d63b6