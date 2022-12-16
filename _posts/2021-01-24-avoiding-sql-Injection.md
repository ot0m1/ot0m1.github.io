---
layout: post
title: prepared statement を利用してSQLインジェクションを回避する
date: 2021/01/24
author: otomi
tags: [blog]
comments: false
---

[前回の記事](https://akkkky.hatenablog.jp/entry/2021/01/20/001614)で [ruby](http://d.hatena.ne.jp/keyword/ruby)-pg から [CRUD](http://d.hatena.ne.jp/keyword/CRUD) するための方法を書いたが、引数でメモの内容を受け取ってそれをクエリ文字列に結合していたので[SQLインジェクション](http://d.hatena.ne.jp/keyword/SQL%A5%A4%A5%F3%A5%B8%A5%A7%A5%AF%A5%B7%A5%E7%A5%F3)の危険性があった。

そのため prepared statement を利用するように変更した。

<!-- more -->

まず本当に文字列結合だと[SQLインジェクション](http://d.hatena.ne.jp/keyword/SQL%A5%A4%A5%F3%A5%B8%A5%A7%A5%AF%A5%B7%A5%E7%A5%F3)が実行可能なのか、 prepared statement を利用すれば回避できるのかを検証。

参考サイト：[https://www.jpcert.or.jp/java-rules/ids00-j.html](https://www.jpcert.or.jp/java-rules/ids00-j.html)

予めこういうテーブルを作っておく。

```sql
postgres=# CREATE TABLE injection_test (id SERIAL, user_name TEXT NOT NULL, password TEXT NOT NULL);
CREATE TABLE
```

適当にユーザーを追加。 本来はパスワードを平文で保存することはないがテストなのでその辺りは割愛。

```sql
postgres=# SELECT * FROM injection_test;
 id | user_name  |  password
----+------------+------------
  1 | ドラえもん | doradora
  2 | のび太     | nobinobi
  3 | スネ夫     | sunesune
  4 | しずちゃん | shizushizu
(4 rows)
```

こういうコードを書いて…

```ruby
require 'pg'

TABLE_NAME = 'injection_test'
connection = PG.connect(dbname: 'postgres')

# 文字列結合でクエリを実行しちゃってるユーザー認証
def validate_authorization_not_prepared(connection, user_name, password)
  # ユーザー名とパスワードが両方一致するかを確認
  query = "SELECT * FROM #{TABLE_NAME} WHERE user_name = '#{user_name}' AND password = '#{password}'"
  result = connection.exec(query)
  if result.cmd_tuples == 0
    puts 'ログインに失敗しました。'
  else
    result.each { |row| puts "ログインに成功しました。ユーザー情報【id】#{row['id']}【user_name】#{row['user_name']}"}
  end
end
```

認証チェック。

存在するユーザーでかつパスワードも正しければユーザー情報を返し、ユーザー名かパスワードのどちらかが一致しなければ認証に失敗する。

```ruby
validate_authorization_not_prepared(connection, 'ドラえもん', 'doradora')
# => ログインに成功しました。ユーザー情報【id】1【user_name】ドラえもん
validate_authorization_not_prepared(connection, 'のび太', 'nobinobi')
# => ログインに成功しました。ユーザー情報【id】2【user_name】のび太
validate_authorization_not_prepared(connection, 'ジャイアン', 'doradora')
# => ログインに失敗しました。
validate_authorization_not_prepared(connection, 'ドラえもん', 'jaijai')
# => ログインに失敗しました。
```

この段階では意図した挙動ができている。

次に[SQLインジェクション](http://d.hatena.ne.jp/keyword/SQL%A5%A4%A5%F3%A5%B8%A5%A7%A5%AF%A5%B7%A5%E7%A5%F3)を実行してみる。

```ruby
validate_authorization_not_prepared(connection, %q[ドラえもん' OR '1'='1], 'どうせ認証通るからパスワードなんでもいいよね')
# => ログインに成功しました。ユーザー情報【id】1【user_name】ドラえもん
validate_authorization_not_prepared(connection, %q[のび太' OR '1'='1], 'ほんとはジャイアンです')
# => ログインに成功しました。ユーザー情報【id】2【user_name】のび太
```

認証通っちゃった〜！

> _実在する正規のユーザ名であれば、この SELECT 文は validuser のレコードをテーブルから選択する。username='validuser' の評価は真となるので、パスワードはチェックされない。つまり OR 以降の部分は評価されないということである。OR 以降の部分が [SQL](http://d.hatena.ne.jp/keyword/SQL) 言語の式として文法的に正しければ、攻撃者は validuser のレコードへのアクセスを許可される。_

最悪なのがこの攻撃。

```ruby
validate_authorization_not_prepared(connection, 'ジャイアン', %q[' OR '1'='1])
# => ログインに成功しました。ユーザー情報【id】1【user_name】ドラえもん
# => ログインに成功しました。ユーザー情報【id】2【user_name】のび太
# => ログインに成功しました。ユーザー情報【id】3【user_name】スネ夫
# => ログインに成功しました。ユーザー情報【id】4【user_name】しずちゃん
```

なんと全ユーザーの情報が取れる。

> _'1'='1' は常に真なので、このクエリからはテーブル中のすべての行が出力される。ユーザ名とパスワードの検査は行われず、攻撃者は正しいユーザIDやパスワードを知らなくてもログインできてしまう。_

なのでクエリを文字列結合で実行できるようにちゃうと[SQLインジェクション](http://d.hatena.ne.jp/keyword/SQL%A5%A4%A5%F3%A5%B8%A5%A7%A5%AF%A5%B7%A5%E7%A5%F3)が簡単に行えてしまうことが分かった。

次に prepared statement を利用した認証方式。

```ruby
# prepared statement
def validate_authorization_prepared(connection, user_name, password)
  query = "SELECT * FROM #{TABLE_NAME} WHERE user_name = $1 AND password = $2"
  prepare_name = 'validate_authorization'
  delete_if_exist(connection, prepare_name)
  connection.prepare(prepare_name, query)
  result = connection.exec_prepared(prepare_name, [user_name, password])
  if result.cmd_tuples == 0
    puts 'ログインに失敗しました。'
  else
    result.each { |row| puts "ログインに成功しました。ユーザー情報【id】#{row['id']}【user_name】#{row['user_name']}"}
  end
end

def prepare_exist?(connection, prepare_name)
  tuple = connection.exec("SELECT * FROM pg_prepared_statements WHERE name='#{prepare_name}'").cmd_tuples
  tuple.positive?
end

def delete_if_exist(connection, prepare_name)
  connection.exec("DEALLOCATE #{prepare_name}") if prepare_exist?(connection, prepare_name)
end
```

```ruby
validate_authorization_prepared(connection, 'ドラえもん', 'doradora')
# => ログインに成功しました。ユーザー情報【id】1【user_name】ドラえもん
validate_authorization_prepared(connection, 'ジャイアン', 'jaijai')
# => ログインに失敗しました。
validate_authorization_prepared(connection, %q[ドラえもん' OR '1'='1], 'hoge')
# => ログインに失敗しました。
validate_authorization_prepared(connection, 'ジャイアン', %q[' OR '1'='1])
# => ログインに失敗しました。
```

ちゃんとユーザー名とパスワードが一致する場合のみ認証を通せていることが分かる。

というわけで上記を踏まえた、[ruby](http://d.hatena.ne.jp/keyword/ruby)-pg でのセキュアな [CRUD](http://d.hatena.ne.jp/keyword/CRUD) は以下のようになる。

```ruby
class CrudController
  DB_NAME = '任意のデータベース名'
  TABLE_NAME = '任意のテーブル名'

  def initialize
    @connection = PG.connect(dbname: DB_NAME)
    create_table unless table_exist?
  end

  def read_all_note
    read_all_query = "SELECT * FROM #{TABLE_NAME}"
    prepare_name = 'read_all_note'
    delete_if_exist(prepare_name)
    @connection.prepare(prepare_name, read_all_query)
    results = @connection.exec_prepared(prepare_name).map { |result| result }
    results.sort_by { |result| result.values_at('id') }.reverse
  end

  def read_note(id)
    read_note_query = "SELECT * FROM #{TABLE_NAME} WHERE id = $1"
    prepare_name = 'read_note'
    delete_if_exist(prepare_name)
    @connection.prepare(prepare_name, read_note_query)
    @connection.exec_prepared(prepare_name, [id]) { |result| result[0] }
  end

  def create_note(title, body)
    create_note_query = "INSERT INTO #{TABLE_NAME} (title, body) VALUES ($1, $2) RETURNING id"
    prepare_name = 'create_note'
    delete_if_exist(prepare_name)
    @connection.prepare(prepare_name, create_note_query)
    @connection.exec_prepared(prepare_name, [title, body]) { |result| result[0]['id'] }
  end

  def update_note(id, title, body)
    update_note_query = "UPDATE #{TABLE_NAME} SET (title, body) = ($2, $3) WHERE id = $1"
    prepare_name = 'update_note'
    delete_if_exist(prepare_name)
    @connection.prepare(prepare_name, update_note_query)
    @connection.exec_prepared(prepare_name, [id, title, body])
  end

  def delete_note(id)
    delete_note_query = "DELETE FROM #{TABLE_NAME} WHERE id = $1"
    prepare_name = 'delete_note'
    delete_if_exist(prepare_name)
    @connection.prepare(prepare_name, delete_note_query)
    @connection.exec_prepared(prepare_name, [id])
  end

  def id_exist?(id)
    if /^[0-9]+$/.match?(id)
      exist_id_query = "SELECT * FROM #{TABLE_NAME} WHERE id = $1"
      prepare_name = 'id_exist'
      delete_if_exist(prepare_name)
      @connection.prepare(prepare_name, exist_id_query)
      @connection.exec_prepared(prepare_name, [id]).cmd_tuples == 1
    else
      false
    end
  end

  private

  def prepare_exist?(prepare_name)
    tuple = @connection.exec("SELECT * FROM pg_prepared_statements WHERE name='#{prepare_name}'").cmd_tuples
    tuple.positive?
  end

  def delete_if_exist(prepare_name)
    @connection.exec("DEALLOCATE #{prepare_name}") if prepare_exist?(prepare_name)
  end

  def table_exist?
    exist_table_query = "SELECT table_name FROM information_schema.tables WHERE table_name = '#{TABLE_NAME}'"
    prepare_name = 'table_exist'
    delete_if_exist(prepare_name)
    @connection.prepare(prepare_name, exist_table_query)
    @connection.exec_prepared(prepare_name).cmd_tuples == 1
  end

  def create_table
    create_table_query = "CREATE TABLE #{TABLE_NAME} (id SERIAL, title TEXT NOT NULL, body TEXT)"
    prepare_name = 'create_table'
    delete_if_exist(prepare_name)
    @connection.prepare(prepare_name, create_table_query)
    @connection.exec_prepared(prepare_name)
  end
end
```

