---
layout: post
title: VPS で色々遊ぶ①
date: 2023/03/02
author: otomi
tags: [blog, VPS, Apache]
comments: false
---

レンタルサーバーのお仕事をやっていて、色々とサーバーのログを見たりするわけなんですが、ネットでググった断片的な知識で騙し騙しやっている気がしていました。
なのできっちり体系的に学んでいこうと思います。

まずは VPS を借りて、ウェブサーバーをインストールしてサイトを表示させるとことまでやってみます。

<!-- more -->

# サーバーを用意する

まずは遊ぶためのサーバーを用意します。

参考にした本は以下です。

[ゼロからわかる Linux Webサーバー超入門［Apache HTTP Server対応版］ かんたんIT基礎講座](https://www.amazon.co.jp/gp/product/B07HH22LJR/)

VPS は[さくらのVPS](https://vps.sakura.ad.jp/)を使います。
申込みするとことなどは割愛。

OS は CentOS 7 x86_64 を選択。<br>
管理ユーザー名は `root` になる。

初回ログインは`管理ユーザー名@ホスト名 or IPv4 Address`。パスワード認証になっている。

```bash
ssh root@example.vs.sakura.ne.jp
```

サーバーにログインできたらまずはアップデート。

```bash
yum update
```

作業用ユーザーの作成 & 設定
---
root ユーザーの他に作業用のユーザーを作って、通常はそっちのユーザーで作業するのが一般的。

```bash
useradd hoge
passwd hoge
```

sudo を使えるように権限を設定する。

```bash
usermod -G wheel hoge
```

設定ファイルの編集。

```bash
visudo
```

以下の `%wheel  ALL=(ALL)       ALL` の箇所がコメントアウトされているようなら解除する。

```bash
%wheel  ALL=(ALL)       ALL
```

鍵認証の設定
---
SSH のログインがパスワード認証ではセキュリティが低いので、鍵認証に変更する。

公開鍵を置くディレクトリを作成。先程作ったユーザーに切り替えて作業する。

```bash
mkdir ~/.ssh
```

手元のパソコンで以下の作業を行って鍵を作る。2023年現在は暗号方式は ed25519 を使う。

```bash
ssh-keygen -t ed25519
chmod 600 id_rsa.pub
```

サーバーに作った公開鍵をアップロードする。

```bash
scp id_rsa.pub hoge@example.vs.sakura.ne.jp:/home/hoge/.ssh
```

Permission denied となるようであれば一時的に公開鍵のファイルとアップロード先のディレクトリのパーミッションを 777 にする。

ファイルがアップロードできたら手元のパソコンと VPS にアップロードした公開鍵のパーミッションを 600 に変更。及び VPS の公開鍵のファイル名を authorized_keys に変更する。

```bash
cd .ssh
chmod 600 id_rsa.pub
mv id_rsa.pub authorized_keys
```

これで鍵認証での SSH ができるようになっているはずなので、再度ログインし直してみる。

SSHの設定
---

さらにセキュリティを高めるため、root でのログイン及びパスワードでのログインをできないようにする。

root でのログインを禁止する。<br>
ここから先は root 権限での作業となる。先程作ったユーザーでコマンドに `sudo` をつけるか、`sudo -s` を実行して root になっておく。

ssh の設定を変更し、sshd を再起動。

```bash
vim /etc/ssh/sshd_config
# PermitRootLogin yes の行を探してコメントアウトされているならコメント解除して、PermitRootLogin no に変更する root でのログインを禁止
# PasswordAuthentication yes PasswordAuthentication no に変更する パスワード認証の禁止
systemctl restart sshd
```

パスワードでのログインをできないようにする。

ここまでやったらセキュリティが最低限担保されているので、作業を止めていい。お疲れ様でした。
VPS に申し込んでサーバーを起動したら、ここまでは一気にやっておきたい。

インターネットにサーバーを公開したら、攻撃だと疑われるアクセスがマジでガンガンに来ますよ！<br>
もし鍵認証の設定に手間取ったときのために、初回のログインパスワードは必ず強固なものにしておいてください。

# ウェブサーバーを立ち上げる

落ち着いて作業できる状態にしたので、いよいよ Apache をインストールしてウェブサーバーを立ち上げる。

ここからは基本コマンドは `sudo` つけているものとする。

インストール
---

```bash
yum install httpd
```

インストール成功したかを一応確認。

```bash
httpd -v
Server version: Apache/2.4.6 (CentOS)
Server built:   Jan 27 2023 17:36:29
```

Apache を起動する。

```bash
systemctl start httpd
```

起動に成功したら何も表示されない。こちらも一応起動を確認しておく。

```bash
systemctl status httpd
```

`Active: active (running)` が表示されていれば成功。<br>
動いていなければ `Active: inactive (dead)` になる。

自動起動するように設定
---

このままの設定ではサーバーを再起動した際に Apache が自動的に起動してくれない。

```bash
systemctl enable httpd
```

自動起動するようになったかを確認。

```bash
systemctl is-enabled httpd
```

`enabled` となれば自動起動するようになっている。

自動起動をやめたいときは

```bash
systemctl disable httpd
```

これでウェブサーバーは立ち上がったが、さくらVPSの場合は別途作業が必要。

さくらVPSにはパケットフィルターという機能が提供されており、この機能によって Web のポートである TCP 80/443 が閉じている。
これを利用可能にする必要がある。

<img src="https://user-images.githubusercontent.com/6190966/222192265-b855ceac-4d42-4a60-84e0-d624bdd6f882.png">

<img src="https://user-images.githubusercontent.com/6190966/222192279-bbea322e-3d54-47ff-936d-daa745d93d00.png">

ここまでの作業で `http://example.vs.sakura.ne.jp` のようにさくらVPSの URL にアクセスすると無事サイトが表示されているはず。

<img src="https://user-images.githubusercontent.com/6190966/222193643-6fbeab0a-8d8d-4299-9864-864eb9ae23a0.png">

`https` だと証明書のインストールを行っておらずアクセスできないので注意。

コンテンツを設置してみる
---

バーチャルホストを利用していない場合の Apache ドキュメントルートは `/var/www/html/` になる。<br>
このディレクトリにファイルを設置すれば、任意のページを表示させることができる。

このドキュメントルートの設定は `/etc/httpd/conf/httpd.conf` の `DocumentRoot "/var/www/html"` の箇所で変更可能。<br>
設定ファイルを変更した際は Apache の再起動が必要になる。


