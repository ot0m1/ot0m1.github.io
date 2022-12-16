---
layout: post
title: Sinatra で Webアプリケーションを作った
date: 2021/01/16
author: otomi
tags: [blog]
comments: false
---

1年以上ぶりの記事ですね。

去年の12月から[フィヨルドブートキャンプ](https://bootcamp.fjord.jp)に通っており、もう1年以上経ちます。 途中半年以上全然プ[ラク](http://d.hatena.ne.jp/keyword/%A5%E9%A5%AF)ティスをやっていないに等しいような時期があったのですが、ここ2ヶ月くらいはなんとか勉強が習慣化してきたようで連日のようにプ[ラク](http://d.hatena.ne.jp/keyword/%A5%E9%A5%AF)ティスを進めることができている状態です。

[フィヨルド](http://d.hatena.ne.jp/keyword/%A5%D5%A5%A3%A5%E8%A5%EB%A5%C9)ブートキャンプではプ[ラク](http://d.hatena.ne.jp/keyword/%A5%E9%A5%AF)ティスをやった日は必ず日報を書くようになっており、僕も必ず書いています。 その日報をブログにも書いて勉強の過程を公開しようしようと思いつつやれておらず、また過去にさかのぼってブログ書くのもしんどいので中途半端で唐突な始まりですが今日から本気出す。

<!-- more -->

作ったアプリケーションはこれ。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2FAkkkky%2Ffrankly-note" title="Akkkky/frankly-note" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"></iframe><cite class="hatena-citation"><a href="https://github.com/Akkkky/frankly-note">github.com</a></cite>

## REST のリソース設計は思っているより全然難しい

提出物を作成した段階ではわりとスムーズに進められたな〜とか思っていたが、レビューをいただくと色々考慮漏れがあることが分かってきた。

- リソース名の単数形と複数形を統一する
- [URI](http://d.hatena.ne.jp/keyword/URI) の構造を統一する
- 実際にブラウザからアクセスするときの URL を考える
  - ブラウザからアクセスしたときに分かりにくいリソース表現になっていないか
- 実際にブラウザからアクセスするときに区別できない URL になっていないか
  - 異なるリソースを同じ URL で表現しようとしていないか
  - 安易に id だけでリソースを区別しようとするとブラウザから見たときに人間が区別できない

## .DS\_Store に困らせられた

データベースで管理せずにファイルにデータを保存する関係でファイル操作を行うためか、今までは特に気にしていなかった .DS\_Store ファイルがいつの間にかできており、うっかりリモー[トリポジ](http://d.hatena.ne.jp/keyword/%A5%C8%A5%EA%A5%DD%A5%B8)トリへ追加してしまうことがあった。

今までの経験から安易に `rm` したり、リモー[トリポジ](http://d.hatena.ne.jp/keyword/%A5%C8%A5%EA%A5%DD%A5%B8)トリ側で消しちゃうと差分に困るのでしっかり `git rm` できたのは良かった。 .gitignore に追記することでリモー[トリポジ](http://d.hatena.ne.jp/keyword/%A5%C8%A5%EA%A5%DD%A5%B8)トリに追加されることは回避できる。

## 空の[ディレクト](http://d.hatena.ne.jp/keyword/%A5%C7%A5%A3%A5%EC%A5%AF%A5%C8)リは `git add` できない

メモを格納する空の[ディレクト](http://d.hatena.ne.jp/keyword/%A5%C7%A5%A3%A5%EC%A5%AF%A5%C8)リを作成していたのだが、リモー[トリポジ](http://d.hatena.ne.jp/keyword/%A5%C8%A5%EA%A5%DD%A5%B8)トリ側に反映せずにちょっと困った。

そのような場合は .gitkeep ファイルを置いたりするらしい。[https://qiita.com/tommy\_aka\_jps/items/b2ae85cbeab77e12a925](https://qiita.com/tommy_aka_jps/items/b2ae85cbeab77e12a925)

これで無事にリモー[トリポジ](http://d.hatena.ne.jp/keyword/%A5%C8%A5%EA%A5%DD%A5%B8)トリへ空[ディレクト](http://d.hatena.ne.jp/keyword/%A5%C7%A5%A3%A5%EC%A5%AF%A5%C8)リを追加できた。

## Rubocop の言うことをちゃんと聞く

提出直前になって rubocop を通してみたときにセキュリティ関連の警告が出たが、これはデータベースを利用せずにメモをファイルで管理しているために表示されるものであり、今回の実装ではしょうがないと思いこんでいた。

のでそのまま提出しようとしたが、ファイルを読み込むだけで警告が出るはずないだろう、自分よりはるかに経験豊富な人達が作ってくれているライブラリなのに安易にしょうがないで片付けるのは早とちりなのではないかと思い、ちゃんと警告文と向き合った。

表示されていた警告文は以下の2つ。

```ruby
lib/crud_controller.rb:62:3: C: Security/Open: The use of Kernel#open is a serious security risk.
  open(file_path, 'w') do |file|
  ^^^^
```

```ruby
lib/crud_controller.rb:17:26: C: [Correctable] Security/JSONLoad: Prefer JSON.parse over JSON.load.
      json_object = JSON.load(json_file)
                         ^^^^
```

- `open` メソッドはOSコマンドインジェクションの[脆弱性](http://d.hatena.ne.jp/keyword/%C0%C8%BC%E5%C0%AD)があるらしい[https://www.lanches.co.jp/blog/5996](https://www.lanches.co.jp/blog/5996)`File.open` を使うことで存在しないパスの場合はエラーになるのでOSコマンドインジェクション疑われるようなファイルのパス以外の文字列が渡されてもエラーにして回避できる。
- `JSON.load` で直接ファイルを開かない`JSON.parse`を推奨される理由が自分の検索能力では調べきれなかったが、なるべくファイルは開かないに越したことはないからなかと推測した。 そのため`File.open` で読み込んだファイルオブジェクトを `read` して文字列を渡して `parse` してもらうように変更した。

    json\_object = JSON.parse(json\_file.read)

上記を行って警告は全解消できた。 と同時に、これ実務でもし修正せずにそのまま本番環境に適用させたら[脆弱性](http://d.hatena.ne.jp/keyword/%C0%C8%BC%E5%C0%AD)を作ったことになるなぁと怖くなった。

警告文を流し読みせず、しっかり向き合うように改めて思えるいい機会だった。

# 感想など

[Sinatra](http://d.hatena.ne.jp/keyword/Sinatra) のプ[ラク](http://d.hatena.ne.jp/keyword/%A5%E9%A5%AF)ティスの提出物を完成させられた。

ところどころハマったところはあったが、かつてのように何が分からないかも分からないのでハマりを解消する手がかりさえ全く不明、というような頭を抱えるようなドハマリはなく、自分で調べて解消できるようになっていたので成長しているのかなと感じた。

