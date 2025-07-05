---
title: "【Faraday】multipartファイル送信で「Hash into String」を解決"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails", "ruby", "faraday", "multipart"]
published: true
---

## はじめに

Faradayで外部APIにファイルをmultipart形式でPOSTしようとしたところ、`no implicit conversion of Hash into String` というエラーが発生しました。
今回はこのエラーを解決します。

## 前提

Faradayでファイル付きPOSTに必要なgemをインストールしています。

```ruby
# Gemfile
gem 'faraday'
gem 'faraday-multipart'
```

```bash
$ bundle install
```

## エラーが発生したコード

```ruby
# transcription_fileには.vtt形式の文字起こしデータを想定
transcription_file = Tempfile.new(['transcription', '.vtt'])

faraday = Faraday.new(url: 'https://api.example.com') do |f|
  f.request :multipart  # multipart/form-data形式でファイルを送信するように指定
  f.request :url_encoded
  f.adapter Faraday.default_adapter
end
faraday.post('/upload_transcription') do |req|
  req.body = {
    file: Faraday::Multipart::FilePart.new(transcription_file, 'text/plain')
  }
end
```

## 発生したエラー

```bash
TypeError (no implicit conversion of Hash into String)
```
`Hash` を `String` に暗黙的に変換しようとして失敗したときに出るエラーが発生。

## 解決方法

以下のように修正することで、正常にmultipart形式でファイルを送信できるようになりました🎉

```ruby
# Faradayインスタンスの初期化
faraday = Faraday.new(url: 'https://api.example.com') do |f|
  f.request :multipart
  f.request :url_encoded
  f.adapter Faraday.default_adapter
end
# payloadはFaradayインスタンスの外で定義
payload = {
  file: Faraday::Multipart::FilePart.new(
    transcription_file,
    'text/plain'
  )
}
# POST時にpayloadを第2引数で渡す
response = faraday.post('/upload_transcription', payload)
```

## エラーによる学び

Faradayではブロック式`req.body = {...}` を避け、 `post(path, payload)` の形で渡す方が安全であることがわかりました。

## 参考記事

https://github.com/lostisland/faraday-multipart
