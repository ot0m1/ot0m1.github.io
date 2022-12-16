---
layout: post
title: ruby-pg を触ってみる
date: 2021/01/18
author: otomi
tags: [blog]
comments: false
---

[Sinatra](http://d.hatena.ne.jp/keyword/Sinatra)アプリでのデータ保存先をデータベースに変えるため、[ruby-pg](https://github.com/ged/ruby-pg)を触ってみる。 簡単な Ruby スクリプトを書いて「書き込み」「読み取り」「編集」「削除」の一連の操作ができるようにする。

<!-- more -->

まずはTableの作成。

- Ruby側

```ruby
require 'pg'

connection = PG.connect( dbname: 'postgres' )
connection.exec('CREATE TABLE sinatra_table (id INTEGER PRIMARY KEY, title TEXT NOT NULL, body TEXT)')
```

- PostgreSQL側

```sql
-> % ruby test.rb 
-> % psql -U aksk postgres
psql (13.1)
Type "help" for help.

postgres=# \d
           List of relations
 Schema |     Name      | Type  | Owner
--------+---------------+-------+-------
 public | sinatra_table | table | aksk
(1 row)

postgres=#
```

Tableできた。が、これって`postgres`っていうデータベースが存在することが予め分かっているのでコネクションを張れているが、自作の[Sinatra](http://d.hatena.ne.jp/keyword/Sinatra)のWebアプリケーションをcloneしてもらって他の人のパソコン上で動かすときはデータベース名どうやって決めればいいのだろうか？

`PG::Connection.new(host, port, options, tty, dbname, user, password)` これで任意のホストへ接続できるので、自分が契約している[VPN](http://d.hatena.ne.jp/keyword/VPN)へコネクション張れば誰のパソコン上でも予めわかっているデータベース名で接続できるけど、そういうことじゃないよな〜…。

