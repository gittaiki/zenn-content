---
title: "【Part2】ActiveRecordから学ぶSQL"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails", "ruby", "sql"]
published: true
---

## はじめに

モデルのレコード数や、合計値を求めるActiveRecordで、どのようなSQLが実行されるのか確認し、SQLの基礎を固める

※以前作成した記事の続きです👇
https://zenn.dev/y_taiki/articles/learn_sql_from_activerecord

## モデル定義

:::details 内容
    
UserとArticleの2つのモデルを作成します。

User：Articleは1：多で関連付けます。

### Userモデル

```ruby
# user.rb
class User < ApplicationRecord
  has_many :articles, dependent: :destroy
end
```

### Articleモデル

```ruby
# article.rb
class Article < ApplicationRecord
  belongs_to :user

  enum status: { private: 1, public: 2 }
end
```
:::

## テーブル設計

usersとarticlesテーブルの各カラムとレコードは以下になります👇

### usersテーブル

| id | name |
| --- | --- |
| 5 | 夏目漱石 |
| 6 | 芥川龍之介 |

### articlesテーブル

| id | user_id | title | price | status |
| --- | --- | --- | --- | --- |
| 1 | 5 | 吾輩は猫である | 693 | 1(”private”) |
| 2 | 5 | 坊っちゃん | 286 | 2(”public”) |
| 3 | 5 | こころ | 407 | 1(”private”) |
| 4 | 6 | 羅生門 | 440 | 2(”public”) |
| 5 | 6 | 蜘蛛の糸 | 294 | 1(”private”) |

## ****ActiveRecord****

### count(カラム名)

```bash
pry(main)> Article.count(:id)
=> 5
```

```sql
SELECT COUNT("articles"."id") FROM "articles"
```

`SELECT COUNT("articles"."id")`：`articles`テーブルのidカラムの`レコード数`をカウントする

### sum(カラム名)

```bash
pry(main)> Article.sum(:price)
=> 2120
```

```sql
SELECT SUM("articles"."price") FROM "articles"
```

`SELECT SUM("articles"."price")`：`articles`テーブルのpriceカラムの値の`合計`を計算する

### average(カラム名)

```bash
Article.average(:price).to_f
=> 424.0
```

```sql
SELECT AVG("articles"."price") FROM "articles"
```

`SELECT AVG("articles"."price")`：`articles`テーブルのpriceカラムの値の`平均`を計算する

### left_joins(関連名)

```bash
pry(main)> User.left_joins(:articles)
```
:::details 実行結果
```bash
=> [#<User:0x00007ff22a5e25a8
  id: 5,
  name: "夏目漱石",
  email: "souseki@example.com",
  created_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00>,
 #<User:0x00007ff22a5e24e0
  id: 5,
  name: "夏目漱石",
  email: "souseki@example.com",
  created_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00>,
 #<User:0x00007ff22a5e2418
  id: 5,
  name: "夏目漱石",
  email: "souseki@example.com",
  created_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00>,
 #<User:0x00007ff22a5e2350
  id: 6,
  name: "芥川龍之介",
  email: "akutagawa@example.com",
  created_at: Sun, 17 Sep 2023 01:37:38 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:37:38 UTC +00:00>,
 #<User:0x00007ff22a5e2288
  id: 6,
  name: "芥川龍之介",
  email: "akutagawa@example.com",
  created_at: Sun, 17 Sep 2023 01:37:38 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:37:38 UTC +00:00>]
```
:::

```sql
SELECT "users".* FROM "users" LEFT OUTER JOIN "articles" ON "articles"."user_id" = "users"."id"
```

`LEFT OUTER JOIN "articles"`：2つのテーブル間の結合を行い、
左側は`FROM`で指定した`users`テーブルのすべてのレコード、
右側は`JOIN`で指定した`articles`テーブルから、マッチするレコードを選択する。
つまり、ユーザーが記事を持っていない場合でも、そのユーザーの情報は結果に含まれる。

## おまけ

`RIGHT JOIN` をサポートするActiveRecord はありませんでした。
以下のコードをRailsコンソールで実行すると`RIGHT JOIN`できました。

```bash
User.joins("RIGHT JOIN articles ON articles.user_id = users.id")
```
:::details 実行結果
```bash
=> [#<User:0x00007ff228ffabc8
  id: 5,
  name: "夏目漱石",
  email: "souseki@example.com",
  created_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00>,
 #<User:0x00007ff228ffaad8
  id: 5,
  name: "夏目漱石",
  email: "souseki@example.com",
  created_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00>,
 #<User:0x00007ff228ffaa10
  id: 5,
  name: "夏目漱石",
  email: "souseki@example.com",
  created_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:35:43 UTC +00:00>,
 #<User:0x00007ff228ffa920
  id: 6,
  name: "芥川龍之介",
  email: "akutagawa@example.com",
  created_at: Sun, 17 Sep 2023 01:37:38 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:37:38 UTC +00:00>,
 #<User:0x00007ff228ffa808
  id: 6,
  name: "芥川龍之介",
  email: "akutagawa@example.com",
  created_at: Sun, 17 Sep 2023 01:37:38 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:37:38 UTC +00:00>]
```
:::

```sql
SELECT "users".* FROM "users" RIGHT JOIN articles ON articles.user_id = users.id
```

## 参考記事

https://zenn.dev/y_taiki/articles/learn_sql_from_activerecord
