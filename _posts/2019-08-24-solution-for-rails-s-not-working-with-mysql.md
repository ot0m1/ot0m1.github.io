---
layout: post
title: Ruby on Rails でデータベースを mysql で rails s できない場合の解決法
date: 2019/08/24
author: otomi
tags: [blog]
comments: false
---

[前回](https://akkkky.hatenablog.jp/entry/2019/08/22/222546)の続き。

<!-- more -->

前回

`$ rails new i_want_to_use_mysql -d mysql` ば無事成功したが、今度は `rails s` で `ActiveRecord::NoDatabaseError` が発生した。

![f:id:a_k_s_h:20190824063458p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/a/a_k_s_h/20190824/20190824063458.png "f:id:a\_k\_s\_h:20190824063458p:plain")

データベースが作られていないことによるエラー。 データベースを作成する必要がある。

    rails db:create

ちなみに予め[mysql](http://d.hatena.ne.jp/keyword/mysql)を起動しておく必要があります！

    $ mysql.server start Starting MySQL SUCCESS!

Rails5.0未満のときは `rake` というコマンドだったらしい。

