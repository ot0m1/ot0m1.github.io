---
layout: post
title: VPS で色々遊ぶ③
date: 2023/03/05
author: otomi
tags: [blog, VPS, Apache]
comments: false
---

[前回](https://blog.otomi.world/play-around-with-vps-2/)の続き。
3回目やっていきましょう。

今回も[ゼロからわかる Linux Webサーバー超入門［Apache HTTP Server対応版］ かんたんIT基礎講座](https://www.amazon.co.jp/gp/product/B07HH22LJR/)を参考にさせていただいております。

<!-- more -->

# Basic 認証と Digest 認証を設定する

Apache では .htaccess を使った Basic 認証と Digest 認証に対応している。<br>
どちらもサーバーにパスワードのファイルを置いてそれを認証に利用する仕組み。

## .htaccess を有効にする

Basic 認証と Digest 認証を有効にするには、Apache の .htaccess を有効にする必要がある。
設定ファイルで設定を変更する。

設定ファイル（`/etc/httpd/conf/httpd.conf`）の `<Directory "/var/www/html">` 以下に `AllowOverride None` となっている箇所の None を All に書き換える。<br>
設定を変更したら反映させるためにわすれずに `systemctl reload httpd` を実行。

## パスワードファイルを作る

ユーザー名とパスワードの組み合わせが記述されたパスワードファイルをサーバーに置き、入力された値と一致するかで認証する。<br>
パスワードファイルは Basic 認証の場合 .htpasswd、Digest 認証の場合は .htdigest という名称にするのが一般的。

パスワードファイルにはパスワードがそのまま記載されるわけではない。ハッシュ化されたものが保存される。<br>ハッシュ化は非可逆なので、仮にハッシュ化されたパスワードが漏洩しても元のパスワードは解読されない（解読されないという記述だけだと色々と考慮漏れがあるけど、この記事の趣旨をはずれるので割愛する）。

Basic 認証における .htpasswd ファイルの書式は「ユーザー名」と「ハッシュ化されたパスワード」をコロンでつないで記述する。<be>Digest 認証の場合はユーザー名に「レルム」と「ユーザー名とレルムとパスワードをコロンでつなげて MD5 でハッシュ化した値」をつなげて記述する。レルムとはユーザー名とパスワードを入力する画面で表示される文言のこと。

例えば以下のような「ユーザー名とレルムとパスワードだとする。

|  ユーザー名  |  パスワード  |  レルム  |
| ---- | ---- | ---- |
|  otomi  |  otomipass  |  Input your password  |

```txt
otomi:Input your password:2ce1a7a69b30bf92f1df25fe0c1f5651
```

パスワードをハッシュ化する
---

パスワードをハッシュ化するのに htpasswd コマンドという Apache で提供されている機能を使うことができる。

```bash
htpasswd -c /var/www/html/.htpasswd otomi
New password: 
Re-type new password: 
Adding password for user otomi
```

ファイルを入力するときは `-c` オプションをつける。引数はファイルのパス、ユーザー名。

```
cat .htpasswd 
otomi:$apr1$dD.A2o9q$M97KWO/zxVnTTmOAjDJ7e/
```

確かにハッシュ化されている。

ユーザーとパスワードを設定する
---

パスワードファイルが準備できたら制限をかけたいディレクトリに .htaccess ファイルを置いて認証機能を有効にする。

アクセス制限を実施したいディレクトリに .htaccess を設置する。<br>記述は以下のようになる。

```bash
AuthType 認証のタイプ
AuthName "メッセージとして表示したい文字 / 半角英数字のみ"
AuthUserFile .htpasswd ファイルのパス
Require valid-user
```

- AuthType : Basic 認証であれば `Basic` Digest 認証であれば `Digest`
- AuthUserFile : Digest 認証のときは AuthDigestFile になる

設定して無事 Basic 認証がかかった。

<img src="https://user-images.githubusercontent.com/6190966/222945639-b54d0716-99c6-4d24-9d0f-175b03f66eec.png">

Basic 認証だと以下のようなサンプルになる。

```bash
AuthType Basic
AuthName "Input your password"
AuthUserFile /var/www/html/.htpasswd
Require valid-user
```

Digest 認証をやってみる
---

Digest 認証は Basic 認証より手順が多い。

まずはパスワードファイルの作成。パスワードファイルの作成は `htpasswd` ではなく `htdigest` を使う。レルムの引数が追加されている。

```bash
htdigest -c /var/www/html/.htdigest "Input your password" otomi
Adding password for otomi in realm Input your password.
New password: 
Re-type new password: 
cat .htpasswd 
otomi:Input your password:350a9526c370d3b87b798407a2ee5086
```

.htaccess は以下のような感じになる。

```bash
AuthType Digest
AuthName "Input your password"
AuthUserFile /var/www/html/.htdigest 
Require valid-user
```

# PHP を動かせるようにする

Apache でプログラム言語を実行するには2つの方法がある。

モジュール
---

Apache にはプログラムの実行機能がないので、モジュールが行う。そのためプログラム言語ごとのモジュールをインストールしたり Apache の設定を行ったりする。<br>
基本的には該当するモジュールをインストールして拡張子と結びつけておく。そうすると、閲覧者のブラウザから該当の拡張子のファイルが要求されたときにプログラムが実行されその結果が返されるようになる。

モジュールの実行はディレクトリ単位で設定することもでき、「このディレクトリではプログラムの実行を許可するが、このディレクトリでは許可しない」といったこともできる。

CGI
---

どんなプログラム言語でも対応できる方法として CGI（Common Gateway Interface）という仕組みがある。モジュールでプログラムを読みこんで実行するのではなく、実行形式のファイルを実行する仕組み。この方法であれば Apache がモジュールとして対応していない言語も実行できる。

例えば Apache で Perl を実行する際にモジュールを使う場合であれば、そのモジュールが Perl のソースファイルを読み込んで、それを解釈して実行する。それに対して CGI では一行目に「Perl インタプリタ」という Perl の実行エンジンとなるプログラムを指定しておくと、このインタプリタが読み込んで実行する。

前者は Apache と一体になって動くのに対し、後者はクライアントからアクセスされるたびに Perl インタプリタを実行する必要があり効率が悪く遅いため近年ではあまり使われない。

## PHP の環境を整える

PHP の実行環境には mod_php というモジュールと Apache の設定変更が必要。

```bash
yum install php
```

モジュールをインストールすると `/etc/httpd/conf.d` の中に `php.conf` という PHP の設定ファイルが作られる。この中にある `SetHandler application/x-httpd-php` の箇所が `.php` の拡張子のファイルと PHP モジュールを紐付けている部分。

また、`/etc/httpd/conf.modules.d` ディレクトリに `10-php.conf` というモジュールの設定ファイルもできる。
