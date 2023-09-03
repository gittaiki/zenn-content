---
title: "Action Mailerでメール送信機能を実装" # 記事のタイトル
emoji: "🐈" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails", "ruby"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## Action Mailerとは？

メール送信機能を実装できる

## 実装してみる

今回はユーザー登録成功時、入力したメールアドレス宛に登録完了メールを送信する

### ユーザー登録機能作成

```ruby
$ rails g scaffold user name email
$ rails db:migrate
```

### Action Mailer用ファイルを生成

以下を実行すると、app/mailers配下にファイルが生成
またapp/views配下に、メール内容を編集できるHTMLとテキストファイルが生成

```ruby
$ rails g mailer UserMailer
```

### コントローラーを編集

・メソッドの説明
`with`：引数(key: value)でパラメーターを渡す
`deliver_now`：同期的
`deliver_later`：非同期的（※Sidekiq等の設定が必要）
```ruby
# users_controller.rb
def create
  @user = User.new(user_params)

  respond_to do |format|
    if @user.save
      # UserMailerのwelcomメソッドを発火
      UserMailer.with(email: @user.email, name: @user.name).welcome.deliver_now

      format.html { redirect_to @user, notice: 'ユーザー登録が完了しました。' }
      format.json { render :show, status: :created, location: @user }
    else
      format.html { render :new }
      format.json { render json: @user.errors, status: :unprocessable_entity }
    end
  end
end

private
  def user_params
    params.require(:user).permit(:name, :email)
  end
```

### メイラーを編集

app/mailers配下に生成されたメイラーファイルに、送信用メールアドレス等を設定

```ruby
# mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: 'rails@example.com' # 送信用メールアドレス
  layout 'mailer' # views/layout配下のmailer.(htmlかtext).erbを指定
end
```

```ruby
# mailers/user_mailer.rb
class UserMailer < ApplicationMailer
  def welcome(email:, name:)
    @name = name
    # mailメソッドで、HTMLとテキストファイルがあるか探す
    mail(to: email, subject: '登録完了')
  end
end
```

views/user_mailer配下のHTMLとテキストファイルに、送信する内容を編集

```ruby
<p> <%= @name %> 様</p>
<p> ユーザー登録が完了しました。</p>
```

### 開発環境でメールが送信されるか確認

開発環境で送信したメールを確認できる`letter_opener_web`をbundle install

```ruby
# Gemfile
group :development do
  gem 'letter_opener_web'
end
```

```ruby
# routes.rb
Rails.application.routes.draw do
  mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.development?
  # 省略
end
```

```ruby
# config/environments/development.rb
Rails.application.configure do
  # 省略
  config.action_mailer.delivery_method = :letter_opener_web
end
```

ユーザー登録後、[http://localhost:3000/letter_opener](http://localhost:3000/letter_opener%E3%81%AB%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9)
にアクセスし、メールが送信されていれば実装完了🎉

## 参考記事

https://qiita.com/annaaida/items/81d8a3f1b7ae3b52dc2b

https://github.com/fgrehm/letter_opener_web

https://techtechmedia.com/letter_opener_web/