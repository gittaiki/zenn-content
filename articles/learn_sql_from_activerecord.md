---
title: "ActiveRecordから学ぶSQL"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails", "ruby", "sql"]
published: true
---

## はじめに

個人的によく見るActiveRecordで、どのようなSQLが実行されるのか確認し、SQLの基礎を固める

## モデル定義

UserとArticleモデルを作成します。
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

## テーブル設計

usersとarticlesテーブルの各カラムとレコードは以下になります👇

### usersテーブル

| id | name |
| --- | --- |
| 5 | 夏目漱石 |
| 6 | 芥川龍之介 |

### articlesテーブル

| id | user_id | title | status |
| --- | --- | --- | --- |
| 1 | 5 | 吾輩は猫である | 1(”private”) |
| 2 | 5 | 坊っちゃん | 2(”public”) |
| 3 | 5 | こころ | 1(”private”) |
| 4 | 6 | 羅生門 | 2(”public”) |
| 5 | 6 | 蜘蛛の糸 | 1(”private”) |

## ActiveRecord

RailsコンソールでActiveRecordを実行し、生成されるSQLを見ていきます。
※SQL以外の実行結果（=>以降）の説明は割愛

### pluck

```bash
pry(main)> User.pluck(:name)
=> ["夏目漱石", "芥川龍之介"]
```

```sql
SELECT "users"."name" FROM "users"
```

`SELECT "users"."name"`：`users`テーブルのnameカラムを選択
`FROM "users"`：`users`テーブルを取得

### where

```bash
pry(main)> Article.where(user_id: '5')
```
:::details 実行結果
```bash
=> [#<Article:0x00007ff11c013050
  id: 1,
  user_id: 5,
  title: "吾輩は猫である",
  status: "private",
  created_at: Sun, 17 Sep 2023 01:49:30 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:30 UTC +00:00>,
 #<Article:0x00007ff11c012cb8
  id: 2,
  user_id: 5,
  title: "坊っちゃん",
  status: "public",
  created_at: Sun, 17 Sep 2023 01:49:35 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:35 UTC +00:00>,
 #<Article:0x00007ff11c012b00
  id: 3,
  user_id: 5,
  title: "こころ",
  status: "private",
  created_at: Sun, 17 Sep 2023 01:49:38 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:38 UTC +00:00>]
```
:::

```sql
SELECT "articles".* FROM "articles" WHERE "articles"."user_id" = ?  [["user_id", 5]]
```

`WHERE "users"."id" = ?`：`users`テーブルの`id`カラムの値が、後続のバインド変数`["id", 5]`と等しいレコードだけを取得

### where.not

```bash
pry(main)> Article.where.not(user_id: '5')
=> [#<Article:0x00007ff11fa47230
  id: 4,
  user_id: 6,
  title: "羅生門",
  status: "public",
  created_at: Sun, 17 Sep 2023 01:49:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:43 UTC +00:00>,
 #<Article:0x00007ff11fa47140
  id: 5,
  user_id: 6,
  title: "蜘蛛の糸",
  status: "private",
  created_at: Sun, 17 Sep 2023 01:49:47 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:47 UTC +00:00>]
```

```sql
SELECT "articles".* FROM "articles" WHERE "articles"."user_id" != ?  [["user_id", 5]]
```

`WHERE "articles"."user_id" != ?`：`articles`テーブルの`user_id`カラムの値が、後続のバインド変数`["user_id", 5]`と異なるレコードを取得

### find

```bash
pry(main)> User.find(5)
=> #<User:0x00007ff119b308f0
 id: 5,
 name: "夏目漱石",
 省略>
```

```sql
SELECT "users".* FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 5], ["LIMIT", 1]]
```

`LIMIT ?`：結果として返されるレコードの最大数を、後続のバインド変数`["LIMIT", 1]`で制限

### find_by

```bash
pry(main)> User.find_by(name: '夏目漱石')
=> #<User:0x00007ff116303018
 id: 5,
 name: "夏目漱石",
 省略>
```

```sql
SELECT "users".* FROM "users" WHERE "users"."name" = ? LIMIT ?  [["name", "夏目漱石"], ["LIMIT", 1]]
```

`WHERE "users"."name" = ?`：: `users`テーブルの`name`カラムの値が、後続のバインド変数`["name", "夏目漱石"]`と等しいレコードだけを取得

### last

```bash
pry(main)> User.last
=> #<User:0x00007ff1166825b8
 id: 6,
 name: "芥川龍之介",
 省略>
```

```sql
SELECT "users".* FROM "users" ORDER BY "users"."id" DESC LIMIT ?  [["LIMIT", 1]]
```

`ORDER BY "users"."id" DESC`：取得したレコードの`id`より降順に並べ替え

### joins

```bash
pry(main)> Article.joins(:user)
```
:::details 実行結果
```bash
=> [#<Article:0x00007f985c9f76c0
  id: 1,
  user_id: 5,
  title: "吾輩は猫である",
  status: "private",
  created_at: Sun, 17 Sep 2023 01:49:30 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:30 UTC +00:00>,
 #<Article:0x00007f985b3746a0
  id: 2,
  user_id: 5,
  title: "坊っちゃん",
  status: "public",
  created_at: Sun, 17 Sep 2023 01:49:35 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:35 UTC +00:00>,
 #<Article:0x00007f985b3745d8
  id: 3,
  user_id: 5,
  title: "こころ",
  status: "private",
  created_at: Sun, 17 Sep 2023 01:49:38 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:38 UTC +00:00>,
 #<Article:0x00007f985b374510
  id: 4,
  user_id: 6,
  title: "羅生門",
  status: "public",
  created_at: Sun, 17 Sep 2023 01:49:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:43 UTC +00:00>,
 #<Article:0x00007f985b374448
  id: 5,
  user_id: 6,
  title: "蜘蛛の糸",
  status: "private",
  created_at: Sun, 17 Sep 2023 01:49:47 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:47 UTC +00:00>]
```
:::

```sql
SELECT "articles".* FROM "articles" INNER JOIN "users" ON "users"."id" = "articles"."user_id"
```

`INNER JOIN "users"`:：`users`テーブルと`articles`テーブルを内部結合。INNER JOINは、両方のテーブルに`ON`で指定した「共通のデータ」が存在するレコードのみを返す。

`ON "users"."id" = "articles"."user_id"`：結合する「共通のデータ」を指定。`users`テーブルの`id`と、`articles`テーブルの`user_id`の値が一致するレコードを指定している。

### exists?

```bash
pry(main)> User.exists?(id: 5)
=> true
```

```sql
SELECT 1 AS one FROM "users" WHERE "users"."id" = ? LIMIT ?  [["id", 5], ["LIMIT", 1]]
```

`SELECT 1 AS one`:：`1`を選択し、その数値に`one`という名前（エイリアス）を付けています。レコードがあるかどうかの判定に使いたいだけなので、カラムを指定していない。
SQLはデータの存在を確認するためのもので、取得した結果をもとにRubyが真偽値を返している。

## おまけ

### enum

```bash
pry(main)> Article.find(2).private?
=> false
```

```sql
SELECT "articles".* FROM "articles" WHERE "articles"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
```

enumのヘルパーメソッド`private?` は、SQLを使わず、既にメモリ上にロードされているオブジェクト上で行われる。
そのため、`exists?`同様、取得した結果をもとにRubyが真偽値を返している。

### アソシエーション

```bash
pry(main)> User.find(6).articles
=> [#<Article:0x00007ff11fd1efd0
  id: 4,
  user_id: 6,
  title: "羅生門",
  status: "public",
  created_at: Sun, 17 Sep 2023 01:49:43 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:43 UTC +00:00>,
 #<Article:0x00007ff11fd1ee18
  id: 5,
  user_id: 6,
  title: "蜘蛛の糸",
  status: "private",
  created_at: Sun, 17 Sep 2023 01:49:47 UTC +00:00,
  updated_at: Sun, 17 Sep 2023 01:49:47 UTC +00:00>]
```

```sql
SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT ?  [["LIMIT", 1]]

SELECT "articles".* FROM "articles" WHERE "articles"."user_id" = ?  [["user_id", 5]]
```

アソシエーションを使用すると、新しくSQLが発行・実行される


## 参考記事

https://takakisan.com/sql-exists/
