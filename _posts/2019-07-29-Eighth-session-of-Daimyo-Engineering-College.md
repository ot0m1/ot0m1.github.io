---
layout: post
title: 大名エンジニアカレッジ8回目
date: 2019/07/29
author: otomi
tags: [blog]
comments: false
---
## やったこと

### DBの理論設計の発表

- 対象のサンプルサイトを見て、網羅的にサイトに存在する情報をテーブルに追加してしまっている
- 設計図を完成させることが目的になってしまっている部分があった
- 実際にデータベースを運用するところまで踏み込めて設計できていない
- その結果冗長になっている（もっと正規化できる）
- このデータベースで適切な運用ができるのか、までをイメージして理論設計していきたい

### [Ruby on Rails](http://d.hatena.ne.jp/keyword/Ruby%20on%20Rails) のデプロイ

[ロリポップ！マネージドクラウド](https://mc.lolipop.jp/)への[Ruby on Rails](http://d.hatena.ne.jp/keyword/Ruby%20on%20Rails)のデプロイ。 やっとローカル以外の環境に[Ruby on Rails](http://d.hatena.ne.jp/keyword/Ruby%20on%20Rails)をデプロイできた！ 今まで何度もデプロイに失敗していた（失敗のエラーの原因にもたどり着けなかった）ので素直に嬉しい。

が、全然スムーズではなかった。 なんで環境構築にこんなに苦戦するのか。

ローカルの環境構築のときも大変だったが、リモートへの環境構築はITの包括的な知識が必要とされ、知っておかないといけないことが[Ruby](http://d.hatena.ne.jp/keyword/Ruby)の（入門書にあるようなシンプルな）コードを書くときより爆発的なくらい多い。 ぱっと思いつくだけでもターミナル（シェル）、サーバー、git、公開鍵認証暗号、パッケージ管理システム、データベース、[環境変数](http://d.hatena.ne.jp/keyword/%B4%C4%B6%AD%CA%D1%BF%F4)、[Ruby](http://d.hatena.ne.jp/keyword/Ruby)、[Ruby on Rails](http://d.hatena.ne.jp/keyword/Ruby%20on%20Rails)、などをある程度知っておく必要がある。

それそれの知識を身に着けないと、手順の行程全てが魔法のようになり、応用ができない。完全に同じ状況じゃないと環境構築できなくなってしまう。

なのでこの本を読んだ。

[![「プロになるためのWeb技術入門」 ――なぜ、あなたはWebシステムを開発できないのか](https://images-fe.ssl-images-amazon.com/images/I/61YVe2oD4CL._SL160_.jpg "「プロになるためのWeb技術入門」 ――なぜ、あなたはWebシステムを開発できないのか")](http://www.amazon.co.jp/exec/obidos/ASIN/4774142352/hatena-blog-22/)

[「プロになるためのWeb技術入門」 ――なぜ、あなたはWebシステムを開発できないのか](http://www.amazon.co.jp/exec/obidos/ASIN/4774142352/hatena-blog-22/)

- 作者: 小森裕介
- 出版社/メーカー: [技術評論社](http://d.hatena.ne.jp/keyword/%B5%BB%BD%D1%C9%BE%CF%C0%BC%D2)
- 発売日: 2010/04/10
- メディア: 大型本
- 購入: 57人 クリック: 1,242回
- [この商品を含むブログ (35件) を見る](http://d.hatena.ne.jp/asin/4774142352/hatena-blog-22)

めちゃくちゃ良かった。今までwebを体系的に捉える本数冊読んできたが、これが断然一番いい。[Ruby on Rails](http://d.hatena.ne.jp/keyword/Ruby%20on%20Rails)という[フレームワーク](http://d.hatena.ne.jp/keyword/%A5%D5%A5%EC%A1%BC%A5%E0%A5%EF%A1%BC%A5%AF)を使うようになるまでのWeb技術の歴史が、どういう問題があり、それを解決するためにどんな技術が開発されたか、という視点で丁寧に説明されてあり、授業で習っていることが線でつながって理解できるようになった。

