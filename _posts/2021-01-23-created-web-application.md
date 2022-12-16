---
layout: post
title: Sinatra で Webアプリケーションを作った際の気付き
date: 2021/01/23
author: otomi
tags: [blog]
comments: false
---

[Sinatra で Webアプリケーションを作った - tmp/log](https://akkkky.hatenablog.jp/entry/2021/01/16/234329) の続き。 レビューいただいた点を修正しての気付きなど。

<!-- more -->

レビューいただいた点について修正にあたって以下の学びがあった。

## HTML エスケープしたときに表示が崩れる

これは当初エスケープに利用している `Rack::Utils.escape_html` 側の出力に原因があると考えていた。そこでどんな処理をしているのかソースを見に行ったら、シンプルに記号を置換しているものであった。

[https://github.com/rack/rack/blob/a05f8d56f9ac4da14dddb8f312a3b43644f73397/lib/rack/utils.rb#L158](https://github.com/rack/rack/blob/a05f8d56f9ac4da14dddb8f312a3b43644f73397/lib/rack/utils.rb#L158)

実際のコード

```ruby
ESCAPE_HTML = {
    "&" => "&amp;",
    "<" => "&lt;",
    ">" => "&gt;",
    "'" => "&#x27;",
    '"' => "&quot;",
    "/" => "&#x2F;"
}

ESCAPE_HTML_PATTERN = Regexp.union(*ESCAPE_HTML.keys)

# Escape ampersands, brackets and quotes to their HTML/XML entities.
def escape_html(string)
    string.to_s.gsub(ESCAPE_HTML_PATTERN){|c| ESCAPE_HTML[c] }
end
```


なので出力される文字列が原因ではないということでもうちょっと自分の実装を追っていたら、単に[エス](http://d.hatena.ne.jp/keyword/%A5%A8%A5%B9)ケープした場合に文字列が長くなりすぎ、HTML の要素からはみ出てしまっているだけだった…。

スタイルで `overflow` を適切に設定していなかったことが原因。幸い15分くらいで原因に思い当たったので良かった。原因が [Ruby](http://d.hatena.ne.jp/keyword/Ruby) のコード側にあると思いこんでいたらドハマリするところだった…。

## 表示に関するコードはコントローラーに該当するファイルに書かない

コントローラーに該当する処理を書いていたファイル内で、一緒に HTML タグを操作していた。 View と Model で担当を分担するため書く場所は分けた方が良い。[MVC](http://d.hatena.ne.jp/keyword/MVC) モデルを意識した実装をする。

## README に [Ruby](http://d.hatena.ne.jp/keyword/Ruby) の推奨バージョンを[自然言語](http://d.hatena.ne.jp/keyword/%BC%AB%C1%B3%B8%C0%B8%EC)で書いていた

`.ruby-version` をコミットしておくとよい。 確かに推奨バージョンを書いて利用者に人力でバージョン指定してもらうより rbenv にやってもらった方が全然いい…！

