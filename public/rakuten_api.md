---
title: rakuten_api
tags:
  - Rails
  - 楽天API
  - 個人開発
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# 【Rails】楽天APIで一度に複数のジャンルIDで検索してみた

## はじめに
Railsでのアプリ作成で楽天APIを使用することになったのですが、その際に一度の検索で複数のジャンルIDを使用した検索機能を実装しました。使用機会は多くないものだと感じましたが、記事にまとめたいと思います。


※このページの情報は2024年3月時点の情報となります。公式の最新情報などと照らし合わせてお読みください。

## 楽天APIの使用準備

まず、楽天APIの使用準備について簡潔に解説したいと思います。

(1) [Rakuten Web Service](https://webservice.rakuten.co.jp/)のページに移動

(2) 右上にログインボタンや言語選択ボタンがあります。言語を選択してRakuten Web Serviceのアカウントを作成し、ログイン。

(3) ログイン後、言語選択ボタン近くの「アプリID発行」の項目をクリック。新規アプリ登録画面に映るので、必要な項目を入力。

(4) アプリ登録後は「アプリ情報の確認」をクリック。アプリ一覧画面が表示されますが、その項目にあるアプリID(applicationId)が楽天APIを使用する上で必要になります。Rakuten Web Serviceでの作業はこれで終了です。

(5) ここからRailsアプリでの作業です。
Gemfileに次のgemを記載してインストールしましょう。

```ruby
gem 'rakuten_web_service'
gem 'dotenv-rails'
```
アプリIDをGithub上にそのまま上げると悪用される可能性が生じるので、環境変数を設定するgemの使用がおすすめです。今回はdotenv-railsを使用します。

(6) gemのインストールが終わったら、カレントディレクトリで.envファイルを作成し、ファイル内で環境変数を定義します。
```ruby
RWS_APPLICATION_ID='applicationId'
```
applicationIdの部分は、「アプリ情報の確認」ページで確認したアプリIDにしてください。


(7) 楽天APIを使用するためのファイルをconfig/initializersディレクトリ内にファイルを作成します。そして、作成したファイルの内容を次のようにします。

```ruby
RakutenWebService.configure do |c|
  # (必須) アプリケーションID
  c.application_id = ENV['RWS_APPLICATION_ID']

  # (以下の項目は任意で入力)
  # c.affiliate_id = 'YOUR_AFFILIATE_ID' # default: nil
  # c.max_retries = 5 # default: 5
  # c.debug = true # default: false
end
```

最低限必要になるのはc.application_id = ENV['RWS_APPLICATION_ID']の1行です。他は必要があれば追加してください。また、gemの詳しい解説については[公式ドキュメント](https://github.com/rakuten-ws/rws-ruby-sdk/tree/master)を確認してください。

これで楽天APIを使用する準備は完了です。


## 複数のジャンルIDを使用した検索の必要性

そもそも一度の検索で複数のジャンルIDを使用する必要があるかですが、以下の状況で使用を検討できるのではないかと考えています。

(1)特定の商品ジャンルで商品検索を行いたい、かつその商品が複数のジャンルIDに分かれている場合

(2)ユーザーが商品検索を行う際にジャンルを選択する手間を省きたい場合

(3)ユーザーが商品ジャンルを選択する際、探したい商品がどのジャンルに含まれるかわからないことが想定される場合

いずれも実際の開発では稀だと思いますが、遭遇する可能性は少なからず存在すると思います。


## 複数のジャンルIDを使用した検索機能の実装

楽天APIを使用した検索でジャンルIDを設定する場合、次のような形になるかと思います。

```ruby
def search
 if params[:keyword]
  @toothbrushes = RakutenWebService::Ichiba::Item.search(keyword: params[:keyword], 
                                                         genreId: '506384')
 end
end
```

キーワード引数genreIdでジャンルIDを定義することになるかと思いますが、この形だと単一のジャンルIDでしか検索できません。

ではどうやって複数のジャンルIDで検索するかですが、今回私はeachメソッド使用した繰り返し処理での実装を行いました。
```ruby
def search_results
  @results = []
  genre_ids.each do |genre_id|
    results = RakutenWebService::Ichiba::Item.search(keyword: params[:keyword], 
                                                     genreId: genre_id,).to_a
    @results.concat(results)
  end
  p @results
end

def genre_ids
  %w[506385 506386 506387 506389 568329 551692 551693 208522]
end

```

genre_idsメソッドで定義したジャンルIDを取り出して検索し、検索結果を配列にした後、concatメソッドで破壊的に連結するのを繰り返し、最後にpメソッドで検索結果を出力します。

本来であれば、検索結果を格納する配列を用意せずとも配列で返すmapメソッドを使用できれば良いのですが、どうしてもmapメソッドを使用した形に修正できませんでした・・・ここは改善すべき点だと思うので、引き続き調査を行っていきたいと思います。

## まとめ
使用する機会は少ないと思いますが、解説されている記事を発見できなかったので記事にまとめました。執筆するのは今回が初めてで、プログラミング学習も日が浅いので内容的にも技術的にも拙い部分が多いかもしれませんが、こういうやり方もあるのかと何かの参考になれば幸いです。

最後まで読んでいただき、ありがとうございます！