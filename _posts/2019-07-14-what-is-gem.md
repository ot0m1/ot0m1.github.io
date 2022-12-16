---
layout: post
title: Homebrew/rbenv/gem/Bundler ってなんじゃい
date: 2019/07/14
author: otomi
tags: [blog]
comments: false
---
それぞれどう違うのか説明できないですよね。 僕は説明できません。 ググってなんとなくてやっていたのでちゃんと理解したいと思います。

## ちゃんと理解するとは

- Qiita（いつもお世話になっています）などの複次情報ではなく、原典にあたる

- 出典元を可能な限り明記する

なお、各々の利用方法の解説はしません。それぞれの違い、なぜ利用するのかを明らかにすることを本記事の目的とします。

<!-- more -->
## 概要

それぞれの公式サイトに書かれていることから、gem以外のHomebrew/rbenv/Bundlerの3つはすべてパッケージ管理システムと言われるものの一種のようです。

> パッケージ管理システムとは、OSというひとつの環境で、各種のソフトウェアの導入と削除、そしてソフトウェア同士やライブラリとの依存関係を管理するシステムである。

[wikipedia](https://ja.wikipedia.org/wiki/%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E7%AE%A1%E7%90%86%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0)より。

### Homebrew

#### 公式サイト

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fbrew.sh%2Findex_ja" title="Homebrew" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"></iframe><cite class="hatena-citation"><a href="https://brew.sh/index_ja">brew.sh</a></cite>

#### 概要　

[brew(1) – The missing package manager for macOS — Homebrew Documentation](https://docs.brew.sh/Manpage)より。

> [brew](http://d.hatena.ne.jp/keyword/brew)(1) – The missing package manager for [macOS](http://d.hatena.ne.jp/keyword/macOS)

_ **package manager** _と書かれています。

> DESCRIPTION  
> Homebrew is the easiest and [most](http://d.hatena.ne.jp/keyword/most) flexible way to install the [UNIX](http://d.hatena.ne.jp/keyword/UNIX) tools [Apple](http://d.hatena.ne.jp/keyword/Apple) didn’t include with [macOS](http://d.hatena.ne.jp/keyword/macOS).

_**説明  
Homebrewは、[Apple](http://d.hatena.ne.jp/keyword/Apple)が[macOS](http://d.hatena.ne.jp/keyword/macOS)に含まなかった[UNIX](http://d.hatena.ne.jp/keyword/UNIX)ツールをインストールするための最も簡単で最も柔軟な方法です。**_

ほうほう。[macOS](http://d.hatena.ne.jp/keyword/macOS)にプリインストールされていない[UNIX](http://d.hatena.ne.jp/keyword/UNIX)ツールをインストールするために便利なツールのようですね。

### rbenv

#### 公式サイト

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Frbenv%2Frbenv" title="rbenv/rbenv" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"></iframe><cite class="hatena-citation"><a href="https://github.com/rbenv/rbenv">github.com</a></cite>[Github](http://d.hatena.ne.jp/keyword/Github)のページだけかな。

#### 概要

[GitHub - rbenv/rbenv: Groom your app’s Ruby environment](https://github.com/rbenv/rbenv)より。 README.mdに書かれています。

> Groom your app’s [Ruby](http://d.hatena.ne.jp/keyword/Ruby) environment with rbenv.

_**rbenvを使用してアプリの[Ruby](http://d.hatena.ne.jp/keyword/Ruby)環境を整えます。**_

> Use rbenv to pick a [Ruby](http://d.hatena.ne.jp/keyword/Ruby) version for your application and guarantee that your development environment matches production.

_**rbenvを使用してアプリケーションの[Ruby](http://d.hatena.ne.jp/keyword/Ruby)バージョンを選択し、開発環境と本番環境が一致することを確認してください。**_

> Put rbenv to work with Bundler for painless [Ruby](http://d.hatena.ne.jp/keyword/Ruby) upgrades and bulletproof deployments.

_**苦労のない[Ruby](http://d.hatena.ne.jp/keyword/Ruby)アップグレードと失敗の心配のないデプロイのためにrbenvをBundlerと連携させてください。**_  
お、_Bundler_が出てきましたね。大文字から始まっているので固有名詞のBundlerで間違いなさそうですね。

> One thing well. rbenv is concerned solely with switching [Ruby](http://d.hatena.ne.jp/keyword/Ruby) versions.

_**1つだけです。rbenvは[Ruby](http://d.hatena.ne.jp/keyword/Ruby)のバージョンを切り替えることだけに関心があります。**_

いいですね。[Ruby](http://d.hatena.ne.jp/keyword/Ruby)のバージョンを切り替えることだけに関心があるそうです。こんな風に迷いなく生きたいですね。

### gem

#### 公式サイト

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Frubygems.org%2F" title="RubyGems.org | your community gem host" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"></iframe><cite class="hatena-citation"><a href="https://rubygems.org/">rubygems.org</a></cite>

gemっていうのは[サードパーティ](http://d.hatena.ne.jp/keyword/%A5%B5%A1%BC%A5%C9%A5%D1%A1%BC%A5%C6%A5%A3)のライブラリを指しており、それぞれのgemの公式サイトは載せられない（2019/07 時点で約15万のgemがある・・！）のでgemを管理しているサイトを公式サイトとします。

#### 概要

[Ruby公式サイト](https://www.ruby-lang.org/ja/libraries/)の[ライブラリ](https://www.ruby-lang.org/ja/libraries/)より。

やっと全部日本語のページです・・！

> ライブラリ  
> 多くの[プログラミング言語](http://d.hatena.ne.jp/keyword/%A5%D7%A5%ED%A5%B0%A5%E9%A5%DF%A5%F3%A5%B0%B8%C0%B8%EC)と同様に、[Ruby](http://d.hatena.ne.jp/keyword/Ruby) にも幅広い[サードパーティ](http://d.hatena.ne.jp/keyword/%A5%B5%A1%BC%A5%C9%A5%D1%A1%BC%A5%C6%A5%A3)のライブラリが提供されています。  
>   
> それらのほとんどは “gem” という形式で公開されています。[RubyGems](http://d.hatena.ne.jp/keyword/RubyGems) は ([Ruby](http://d.hatena.ne.jp/keyword/Ruby) に特化した apt-get と同じようなパッケージングシステムで) ライブラリの作成や公開、インストールを助けるシステムです。[Ruby](http://d.hatena.ne.jp/keyword/Ruby) のバージョン 1.9 以降 [RubyGems](http://d.hatena.ne.jp/keyword/RubyGems) は標準添付となっていますが、それ以前のバージョンの [Ruby](http://d.hatena.ne.jp/keyword/Ruby) の場合は自分でインストールする必要があります。

gemというのは[サードパーティ](http://d.hatena.ne.jp/keyword/%A5%B5%A1%BC%A5%C9%A5%D1%A1%BC%A5%C6%A5%A3)で開発されたライブラリのようです。

> [Ruby](http://d.hatena.ne.jp/keyword/Ruby) のライブラリは主に [RubyGems](http://d.hatena.ne.jp/keyword/RubyGems).org に gem として置かれています。直接ウェブサイトを閲覧したり、gem コマンドを使用してそれらを探すことができます。

_**[RubyGems](http://d.hatena.ne.jp/keyword/RubyGems).org**_とは公式サイトとして紹介したサイトですね。

### Bundler

#### 公式サイト

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fbundler.io%2F" title="Bundler: The best way to manage a Ruby application's gems" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"></iframe><cite class="hatena-citation"><a href="https://bundler.io/">bundler.io</a></cite>

#### 概要

> Bundler provides a consistent environment for [Ruby](http://d.hatena.ne.jp/keyword/Ruby) projects by tracking and installing the exact gems and versions that are needed.

_**Bundlerは、必要な正確なgemとバージョンを追跡してインストールすることによって、[Ruby](http://d.hatena.ne.jp/keyword/Ruby)プロジェクトに一貫した環境を提供します。**_

gemが出てきました！

> Bundler is an exit from [dependency](http://d.hatena.ne.jp/keyword/dependency) hell, and ensures that the gems you need are present in development, staging, and production.

_ **Bundlerは依存関係の地獄からの出口であり、あなたが必要とするgemsが開発、ステージング、そしてプロダクションに存在することを保証します。** _

gemsの依存関係を解決するもののようですね。

## 概要を把握したあとで

ここからは僕自身のまとめとなります。 gem以外のHomebrew/rbenv/Bundlerの3つはすべてパッケージ管理システムと言われるものの一種であることが公式サイトの説明から分かりました。

### なぜパッケージ管理システムが必要なのか

環境が複雑になっていくにつれ、それぞのパッケージが人間で管理しきれなくなってくるという問題を解決するためです。

もう一度[wikipedia](http://d.hatena.ne.jp/keyword/wikipedia)の説明を見てみます。

> パッケージ管理システムとは、OSというひとつの環境で、各種のソフトウェアの導入と削除、そしてソフトウェア同士やライブラリとの依存関係を管理するシステムである。

_ **各種のソフトウェアの導入と削除、そしてソフトウェア同士やライブラリとの依存関係を管理する** _とあります。

[Ruby](http://d.hatena.ne.jp/keyword/Ruby)の開発環境だとこういう導入のモチベーションとなります。

- 複数の[Ruby](http://d.hatena.ne.jp/keyword/Ruby)のバージョンで開発や動作テストを行うことになった
- 複数のバージョン管理がしんどい
- rbenvで管理することにする
- rbenvもソフトウェアであり、rbenv自体の管理も必要なのでHomebrewでインストールし、管理する
- ピュアな[Ruby](http://d.hatena.ne.jp/keyword/Ruby)だけで開発はできないのでいろんなgemをインストールする必要がある（[Ruby on Rails](http://d.hatena.ne.jp/keyword/Ruby%20on%20Rails)もgem！）
- ○○のgemをインストールするために○○を先にインストールしておかないという場面が発生した！（いわゆる依存関係）
- gemの管理もしんどい
- Bundlerで管理することにする

それぞれ困った場面があり、それを解決したいから導入するわけですね。

