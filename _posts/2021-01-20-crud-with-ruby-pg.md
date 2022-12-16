---
layout: post
title: ruby-pg で CRUD する
date: 2021/01/20
author: otomi
tags: [blog]
comments: false
---

[ruby-pg を触ってみる](http://localhost:4000/trying-to-touch-ruby-pg/)の続き。

<!-- more -->

[ruby-pg を触ってみる](http://localhost:4000/trying-to-touch-ruby-pg/)の記事で以下のように書いていたが、これは今回のやりたいことのスコープから外れるのでやらない。 予め `sinatra_db` というデータベースが存在すること前提。

> Tableできた。が、これってpostgresっていうデータベースが存在することが予め分かっているのでコネクションを張れているが、自作の[Sinatra](http://d.hatena.ne.jp/keyword/Sinatra)のWebアプリケーションをcloneしてもらって他の人のパソコン上で動かすときはデータベース名どうやって決めればいいのだろうか？

とりあえず CRUD ができるかの最小限のコードを書いてみる。

**このコードは[SQLインジェクション](http://d.hatena.ne.jp/keyword/SQL%A5%A4%A5%F3%A5%B8%A5%A7%A5%AF%A5%B7%A5%E7%A5%F3)が実行可能なので、必ず個人環境でテストするだけに留めてください。  
絶対に個人情報を扱っているような環境に実装するようなことはしないでください。**

以下の記事に[SQLインジェクション](http://d.hatena.ne.jp/keyword/SQL%A5%A4%A5%F3%A5%B8%A5%A7%A5%AF%A5%B7%A5%E7%A5%F3)を回避できるコードを載せているのでこちらも参照ください。

[prepared statement を利用してSQLインジェクションを回避する](http://localhost:4000/avoiding-sql-Injection/)

👇 CRUD ができるかの最小限のコード

```ruby
require 'pg'

# sinatra_db という DB は予め作成しておく

DB_NAME = 'sinatra_db'
TABLE_NAME = 'sinatra_table'

# コネクション作成 
connection = PG.connect(dbname: "#{DB_NAME}")

# table が存在するか確認
def table_exist?(connection)
  exist_table_query = "SELECT table_name FROM information_schema.tables WHERE table_name = '#{TABLE_NAME}'"
  connection.exec(exist_table_query).cmd_status == 'SELECT 1' ? true : false
end

# table 作成
def create_table(connection)
  create_table_query = "CREATE TABLE #{TABLE_NAME} (id SERIAL, title TEXT NOT NULL, body TEXT)"
  connection.exec(create_table_query)
end

# メモの新規作成 RETURNING id で追加した id を返す
def create_note(connection, title, body)
  create_note_query = "INSERT INTO #{TABLE_NAME} (title, body) VALUES ('#{title}', '#{body}') RETURNING id"
  connection.exec(create_note_query) { |result| result[0]['id'] }
end

# メモの編集
def update_note(connection, id, title, body)
  update_note_query = "UPDATE #{TABLE_NAME} SET (title, body) = ('#{title}', '#{body}') WHERE id = #{id}";
  connection.exec(update_note_query)
end

# メモの削除
def delete_note(connection, id)
  delete_note_query = "DELETE FROM #{TABLE_NAME} WHERE id = #{id}";
  connection.exec(delete_note_query)
end
```

[CRUD](http://d.hatena.ne.jp/keyword/CRUD) の各メソッド自体はクエリをそのまま書くようなものだったので割とすぐ実装できたが、table が存在しない場合に追加するための存在確認メソッドを作成したが、table の存在確認をどのように実装するかに時間がかかった。

[PostgreSQL](http://d.hatena.ne.jp/keyword/PostgreSQL) や [ruby](http://d.hatena.ne.jp/keyword/ruby)-pg には引数で table 名を受け取って boolean で存在有無を返すようなメソッドが無いっぽかったので `information_schema.tables` を使ったが、もっとスマートに書けるのかなー。 table の存在確認はググっても様々なやり方が出てくるし、色々な方法で確認できるのだろう。

