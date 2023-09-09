---
title: "[Sidekiq+Redis]非同期的にメール送信する"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails", "ruby", "sidekiq", "redis"]
published: true
---

# Sidekiqとは？

`ActiveJob（ジョブ）`によって作られた一つ一つのタスクが登録されている`Redis（キュー）`を、非同期的にバックグラウンドで実行していくキューアダプター

```perl
# 実行順
Rails（ジョブ） → Redis（キュー） → Sidekiq（キューアダプター）
```

# 使い時は？

`リアルタイム性を伴わない処理`や、`重たい処理`に使われることが多い

該当する処理の具体例👇
- メールの送信
- 画像の処理
- データを集計してCSVに落とす

# 実装目標

以前作成した記事では、Action Mailerで`同期的`にメール送信を行っています。
今回は、`非同期的`にメール送信されるよう実装していきます。

https://zenn.dev/y_taiki/articles/action_mailer#%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%A9%E3%83%BC%E3%82%92%E7%B7%A8%E9%9B%86

# 実装

## SidekiqとRedisの導入

非同期処理に必要なgemをbundle install

```ruby
# Gemfile
gem 'redis'
gem 'redis-namespace'
gem 'sidekiq', '~> 6.4'
```

## 設定

### キューアダプターをSidekiqに設定

```ruby
# config/application.rb
class Application < Rails::Application    
  config.active_job.queue_adapter = :sidekiq # 追加
end
```

### SidekiqとRedisの接続設定

`config/initializers/sidekiq.rb`を作成し、以下のように記載

```ruby
# config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.redis = { url: 'redis://redis:6379', namespace: 'sidekiq' }
end

Sidekiq.configure_client do |config|
  config.redis = { url: 'redis://redis:6379', namespace: 'sidekiq' }
end
```

### Sidekiqのオプション設定

`config/sidekiq.yml`を作成し、以下のように記載

```ruby
# config/sidekiq.yml
:verbose: false
:pidfile: ./tmp/pids/sidekiq.pid
:logfile: ./log/sidekiq.log
:concurrency: 33
:queues:
  - default
```

## メール送信機能を非同期処理化

### コントローラーを編集

https://zenn.dev/y_taiki/articles/action_mailer#%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%A9%E3%83%BC%E3%82%92%E7%B7%A8%E9%9B%86

以上記事のUsersコントローラーで使用している
`deliver_now`メソッドを`deliver_later`メソッドに修正し、非同期でメール送信するようにします

```ruby
# users_controller.rb
def create
  @user = User.new(user_params)

  respond_to do |format|
    if @user.save
      # deliver_nowをdeliver_laterに変更
      UserMailer.with(email: @user.email, name: @user.name).welcome.deliver_later
      
      # 省略
    end
  end
end
```

## ログ確認

メール送信された際、開発サーバーのコンソールログに、以下のようなログがあると実装完了🎉

```perl
# メールがキューに追加されることを示すログ
Enqueued ActionMailer::MailDeliveryJob (Job ID: xxxxx) to Sidekiq(mailers) with arguments: ...
```

```perl
# Sidekiqがキューからジョブを取り出して実行する
[ActiveJob] [ActionMailer::MailDeliveryJob] [Job ID] Performing ActionMailer::MailDeliveryJob from Sidekiq(mailers) with arguments: ..
```

# 参考記事

https://qiita.com/petertakahashi/items/cb9ae73e5ba3020f4a89

https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%81%AE%E5%85%A8%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89