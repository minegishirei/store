インテリジェンスギャザリング

tttttttttttttttttttttttt



# 🏁 この記事の目的

インテリジェンスギャザリングの手段について解説する試みです。

# インテリジェンスギャザイングとは?

インテリジェンスギャザリングとはペネトレーションテストのフェーズの一つで、SNS, 検索エンジン, その他ツールからの情報収集のことを指します。
ここでは一般的に公開されている情報を調査するのが特徴で、通常ここまでの情報収集は攻撃と見做されない。

# Toc


# 🔧 インテリジェンスギャザリングで使用するツール

## 技術調査 : Wappalyzer

まずはそのサイトがどんな技術で動いているかを確認します。

「Wappalyzer - Technology profiler」というChrome拡張機能を使用することで、そのサイトがどんな技術を使用しているかを確認することができます。

<img src="https://lh3.googleusercontent.com/TE5cGjbTbj_mqLFn1_IljQ8NkX8lZZNDJApijpuoug4FMd8g5EsoWjW8ZUcHnlclzo1KknI21_KUmckFNHUE3JCO0w=s1280-w1280-h800">

https://chromewebstore.google.com/detail/wappalyzer-technology-pro/gppongmhjkpfnbhagpmjfkannfbllamg?hl=ja&pli=1

<img src="https://github.com/minegishirei/store/blob/main/security/tools/wappalyzer.png?raw=true">

例えば、上記の内容だと `Node.js` を使用しているので、Nodejsを利用した脆弱性をつけることになります。

このツールを使ってさまざまなサイトをみるといろんなことがわかります。

- 最先端を行っていると思ってた[デジタル庁](https://www.digital.go.jp/)のサイトが意外とphpやjqueryを使っているということ



## キャプチャ取得 : GoFullPage

スクロール全体を取得しなければならない時のツール。

<img src="https://lh3.googleusercontent.com/GP-wB-jQwMAnOn_laoBcMnrX8FKqtrTsGGRYPFivu7iuuLkkC5LLx3rCwMtpFb6P43lJJrI8gRdsjkLJN5HAHfY3ew=s1280-w1280-h800">

インテリジェンスギャザリング時のエビデンスとして有効に使えそう。

https://chromewebstore.google.com/detail/gofullpage-full-page-scre/fdpohaocaechififmbbbbbknoalclacl


## Google Dorking(グーグルドーク)

Google Dorksとはグーグルの検索窓に設定する「検索コマンド」のことです。

標的のサイト上では公表していなくても、 **サーバーの設定ミスなどでサイトに保存されているファイルなどが参照できたりします。**

- `site:xxxx.com keyword` : 指定したサイト内の情報からkeywordの記述があるページを表示する。
    - 具体例 : `site:qiita.com クソアプリ` : SEO上マイナス評価になる可能性のあるタグが出現した。
- `site:xxxx.com filetype:xxx` : 指定したサイト内の情報から「filetype」で指定したファイルの一覧を表示する。
    - `site: filetype:password` : 全てのサイトから、「.password」で終わる拡張子を抜き出すことができる。
- `site:xxxx.com intitle:keyword` : 指定したサイト内の情報からタイトルに「keyword」のページの一覧を表示する。 
    - `site: intitle:admin` : 全てのサイトの情報からタイトルに「admin」のページの一覧を表示する。 
    - `site: intitle:"Django管理サイト"`
    - `site: intitle:"phpMyAdmin"`
- `site:xxxx.com inurl:admin` : 指定したサイト内の情報からurlに「admin」があるページの一覧を表示する。 

もっと知りたい方 -> https://www.exploit-db.com/google-hacking-database
「Google Hacking Database」

いっぱい怪しいコマンドが出てきます。












