---
layout: post
title: rubyのハッシュのキーにスペースが含まれている場合のエラー回避から読み解くSymbolクラスへの変換タイミング
date: 2019/07/04
author: otomi
tags: [blog]
comments: false
---

[ruby](http://d.hatena.ne.jp/keyword/ruby)のハッシュのキーのSymbolクラスへの変換の挙動が面白かったので、備忘録として書く。

<!-- more -->

## 前提

[ruby](http://d.hatena.ne.jp/keyword/ruby)のハッシュのキーにはどんな種類のオブジェクトでも書くことができる。 また、ハッシュの書き方には`{key => value}`と書く場合と`{key: value}`の二種類がある。

`{key => value}`と書くと、キーには入れたオブジェクトがそのまま設定される。`{key: value}`と書くと、キーはSymbolクラスに変換される。

Symbolクラスに変換されるのだが、クラスによってはto\_symメソッドを持っていないので、ハッシュロケットでしか書けない場合がある。

```ruby
# キーがシンボル以外の場合は =>（ハッシュロケット） で書く必要がある
# キーがシンボルの場合は : で書く必要がある
# だが、Stringクラスはto_symメソッドがあるので、 : で書いても暗黙的にSymbolクラスのオブジェクトに変換してくれる

hash1 = {"a" => 1, "b": 2}

hash1.keys.each do |key|
  p "#{key} のクラス=> #{key.class}"
end

# => "a のクラス=> String"
# => "b のクラス=> Symbol"

# Integerクラスはto_symメソッドがないので、この書き方はできない
# syntax error, unexpected ':', expecting => のエラーが発生する
hash2 = {1: 1}

# ハッシュロケットで書く必要がある
hash3 = {1 => 1}
```

文字列aはStringクラスのままだが、文字列bはSymbolクラスのオブジェクトになっていることが分かる。

## シンボルを作成できる文字列

シンボルを作成するにはコロンに続けて変数名やクラス名、メソッド名の識別子として有効な文字列を書く。（要は数字で始まったりハイフンやスペースが含まれていない文字列）

```ruby
# OK
:This
:is
:_a
:pen_4
:$dollar_mark_is_ok
:@at_mark_is_ok
:"hoge" # さらには
:"fuga_piyo" # こんな風に文字列として
:"1234 5678" # シンボルを作成することも可能

# NG
:1 # syntax error, unexpected tINTEGER, expecting tSTRING_CONTENT or tSTRING_DBEG or tSTRING_DVAR or tSTRING_END
:I have a pen # syntax error, unexpected tIDENTIFIER, expecting end-of-input
:I-have-an-apple # `<main>': undefined local variable or method `have' for main:Object (NameError)
```

## 上記から推測されること

ハッシュを作成するときに`{key: value}`と書く場合と、`:String`と書く場合とで同じことが行われている模様。

## 本題

ここでやっと本題。ハッシュのキーにスペースが含まれている場合はどうなるか。

```ruby
# 変数名やクラス名、メソッド名の識別子として有効な文字列ではないのでエラーとなる
hash4 = {this is: "a pen"} # => syntax error, unexpected tLABEL, expecting do or '{' or '('

# to_symメソッドがあるStringクラスのオブジェクトをキーにしているのでSymbolクラスのオブジェクトに変換してくれる
hash5 = {"this is a": "pen", "this is an" => "apple"}

# "this is a"文字列はSymbolクラスのオブジェクトになっているのでStringクラスのオブジェクトでは値がとれない
p hash5["this is a"] # => nil

# "this is an"小文字はStringクラスなので値がとれる
p hash5["this is an"] # => "apple"

# Symbolクラスのオブジェクトに変換することで値がとれる
p hash5[:"this is a"] # => "pen"

# ここでも変数名やクラス名、メソッド名の識別子として有効な文字列ではないのでエラーとなる
p hash5[:this is a] # => syntax error, unexpected tIDENTIFIER, expecting ']'
```

## 調べた結論

`:String`と書いてシンボルを作成する場合や、ハッシュを作成するときに`{key : value}`と書く場合、`key.to_sym`メソッドが実行されている。

要はどうしても空白が含まれる文字列をSymbolオブジェクトとしてキーに設定したい場合、`{"String": value}`と書いてから変換してもらえばいいちゅうことです。

