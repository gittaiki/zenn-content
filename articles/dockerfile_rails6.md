---
title: "DockerでRails6 + MySQLの環境構築" # 記事のタイトル
emoji: "🐈" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker", "rails", "ruby", "mysql"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 概要
Dockerを使い、Rails6 + MySQLの環境をコンテナ内で構築します。

# Dockerを使うメリット
1. 複数人での開発の際、同じ環境を簡単に構築することができる。
2. 開発環境と同じ本番環境を簡単に作ることができる。

# 環境構築

## Dockerfile作成
Dockerfileでライブラリやパッケージマネージャーをインストールします。

```bash
$ mkdir myapp
$ cd myapp
$ touch Dockerfile
```

```docker:Dockerfile
FROM ruby:2.7.5
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - && apt-get install -y nodejs
RUN npm install --global yarn && npm install -g n && n 16.15.1

WORKDIR /myapp
COPY Gemfile Gemfile.lock /myapp/
RUN bundle install
```

Dockerfile内の環境変数について説明します。

- FROM：ベースイメージを指定。今回はRuby2.7.5を指定しています。
- RUN：コマンド実行。Webpackerを使うため、Yarnパッケージマネージャー（1.x以上）とNode.js（10.13.0以上）をインストールしています。
- WORKDIR：作業ディレクトリを指定。指定したディレクトリが存在していなければ作成されます。
COPY：追加したいファイルを指定したディレクリに追加。

## Gemfile、Gemfile.lock作成
GemfileでRails6をインストールします。
Gemfile.lockは作成するだけで、何も書きません。


```bash
$ touch Gemfile Gemfile.lock
```

```ruby:Gemfile
source 'https://rubygems.org'
gem 'rails', '~> 6.1.7'
```

## Docker image作成
コンテナを作成するためにはDocker imageが必要です。
そのためDockerfileをビルドし、Docker imageを作成します。

```bash
$ docker build .
```

Docker imageが作成されたか以下のコマンドで確認することができます。
```bash
$ docker images
```

## docker-compose.yml作成
作成したDocker imageより`docker run`コマンドでコンテナを作成・起動ができます。
しかし、起動するためには複数のオプションが必要です。
毎回オプションをつけるのは面倒なため、オプションを書き留めることができる
docker-compose.ymlを作成します。
```bash
$ touch docker-compose.yml
```
```yaml:docker-compose.yml
version: '3'

services:
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails server -p 3000 -b '0.0.0.0'"
    ports:
      - "3000:3000"
    volumes:
      - .:/myapp
    tty: true
    stdin_open: true
```

docker-compose.yml内の設定オプションについて一部説明します。

- version：docker-composeのバージョンを指定。
- services：サービスを複数入れるための親のキー。
- web：サービス名。大体webかappが多い。
- command：コンテナ作成時、実行するコマンド。
- build：Dockerfileを含むディレクトリのパス。カレントディレクトリを指定しています。
- ports：公開用のポート。ローカルのポートを指定しています。
- volumes：マウントするパス（場所）を指定。「マウント」とは、ホスト側（コンテナの外）にあるディレクトリ・ファイルを、コンテナの中から利用できるようにすることです。
誰でも起動できるように相対パスで指定しています。

## コンテナ作成
それではコンテナを作成・起動します。
先程作成したdocker-compose.ymlのオプションを使うためには`docker-compose`コマンドを使います。コンテナを起動し続ける（デタッチモード）ため、オプション`-d`をつけます。
```bash
$ docker-compose up -d
```

作成したコンテナ一覧を確認することができます。
STATUSが`UP`だとデタッチモード成功しています。
```bash
$ docker ps
```

## アプリケーション作成
いよいよRailsアプリケーションを作成します。
まず先程作成したコンテナ内に入ります。
```bash
$ docker-compose exec web bash
```
Railsのバージョン、データベースはMySQL、GemfileとGemfile.lockを上書きするよう指定し、rails new！
```bash
$ rails _6.1.7_ new . -d mysql --skip-test --force
```

## database.yml変更
DockerのMySQLに接続するため、rails newで作成された`config/database.yml`にdocker-compose.ymlで指定するデータベースのサービス名`db`を指定します。
```yaml:database.yml
# 省略
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password:
  host: db # 追加
# 省略
```

## docker-compose.ymlに追記
次にデータベース用のコンテナを作成します。
そのため、アプリケーションのコンテナを作成したときと同様、docker-compose.ymlに追記します。サービス名は先程database.ymlで指定した`db`にします。
```yaml:docker-compose.yml
services:
  web:
    # 省略
    stdin_open: true
# ここから下まで追加
    depends_on:
      - db
    volumes:
      - .:/myapp
  db:
    image: mysql
    volumes:
      - db-data:/var/lib/mysql
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'

volumes:
  db-data:
```
docker-compose.yml内の設定オプションについて一部説明します。
- depends_on：依存関係を指定。webはdbと依存関係があり、dbから先にコンテナが作成されます。
- image：タグもしくはイメージ ID の一部を指定。ローカルでもリモートでも指定できます。今回はリモートから取得します。

## データベース用のコンテナ作成
それではデータベース用のコンテナも作成します。
```bash
$ docker-compose up -d
```
作成した2つのコンテナのSTATUSが`UP`だとデタッチモード成功しています。
```bash
$ docker ps
```

## データベース作成
データベースを作成します。
```bash
$ docker-compose exec web bash
$ rails db:create
```
localhost:3000にアクセスすると、Railsのデフォルトページが表示されます🎉

# 参考記事
https://docs.docker.jp/engine/reference/builder.html
https://railsguides.jp/webpacker.html#webpacker%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B
https://docs.docker.jp/v1.9/compose/compose-file.html
https://hub.docker.com/_/mysql
