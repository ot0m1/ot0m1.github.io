---
layout: post
title: VPS で色々遊ぶ②
date: 2023/03/04
author: otomi
tags: [blog, VPS, Apache]
comments: false
---

[前回](https://blog.otomi.world/play-around-with-vps-1/)の続き。

今回も[ゼロからわかる Linux Webサーバー超入門［Apache HTTP Server対応版］ かんたんIT基礎講座](https://www.amazon.co.jp/gp/product/B07HH22LJR/)を参考にさせていただいております。

<!-- more -->

# ファイアウォールの設定を変更する

現在さくらVPSのパケットフィルターという機能でファイアウォールを実現しているので、利用している VPS などに依存しないようにするべく、CentOS 7 が提供しているファイアウォールを利用するようにする。

CentOS 7 ではファイアウォールに Firewalld というソフトウェアが使われている。<br>ファイアウォールの設定集ゾーンには public home work などがある。ゾーンは自分で独自に作ったり、既存のものの設定を変更したりできる。<br>
デフォルトの設定だと public が厳しめ、home が緩め。

まずは起動させる。

```bash
systemctl start firewalld
```

自動起動するように設定する。

```bash
systemctl enable firewalld
```

設定を変更する
---

実際にファイアウォールの設定を変更してみる。<br>
コマンドは `firawall-cmd` というものを使う。

現在のファイアウォールの設定を確認する
---

```bash
firewall-cmd --list-all
```

以下のように表示される。`services` の欄が現在通信を許可しているプロトコル。<br>
デフォルトでは http と https の通信が許可されていないことがわかる。

```
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

ゾーンの設定を変更して http と https を許可する
---

```bash
firewall-cmd --zone=public --add-service={http,https} --permanent
```

複数の service を追加する場合は、{} で囲む。これは firewalld ではなく bash の brace 展開を利用している。
permanent オプションをつけてサーバーが再起動しても変更した設定が有効なままにする。
`success` と表示されたらよい。

このままでは設定が反映されないので、設定の再読み込みを行う。

```bash
firewall-cmd --reload
```

`success` と表示されたらよい。
再度設定を確認してみると、http と https が許可されていることがわかる。

```bash
irewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: dhcpv6-client http https ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

ここまでできたらさくらVPSのパケットフィルターを無効にする。

<img src="https://user-images.githubusercontent.com/6190966/222335409-4d304035-a3aa-4050-87e8-fb5bab5b12dd.png">

# Apache の設定ファイルを編集する

CentOS 7 で yum コマンドを使ってインストールした場合は `/etc/https` ディレクトリに設定ファイルがまとめられている。

```bash
ls -al1 /etc/httpd
合計 20
drwxr-xr-x   5 root root 4096  3月  2 00:43 .
drwxr-xr-x. 86 root root 4096  3月  2 00:43 ..
drwxr-xr-x   2 root root 4096  3月  2 00:43 conf
drwxr-xr-x   2 root root 4096  3月  2 00:43 conf.d
drwxr-xr-x   2 root root 4096  3月  2 00:43 conf.modules.d
lrwxrwxrwx   1 root root   19  3月  2 00:43 logs -> ../../var/log/httpd
lrwxrwxrwx   1 root root   29  3月  2 00:43 modules -> ../../usr/lib64/httpd/modules
lrwxrwxrwx   1 root root   10  3月  2 00:43 run -> /run/httpd
```

Apache の設定や操作を行う場合はこのディレクトリに含まれるファイルを使うことが多い。

|  ディレクトリ名  |  説明  |
| ---- | ---- |
|  conf  |  メインの設定ファイル `httpd.conf` がある  |
|  conf.d  |  追加の設定ファイルがある  |
|  conf.modules.d  |  Apache の追加機能であるモジュールに関する設定ファイルが含まれている  |
|  logs  |  ログを構成するディレクトリ。`/var/log/httpd` へのシンボリックリンク  |
|  modules  |  モジュールファイルを格納するディレクトリ。`/usr/lib64/httpd/modules` へのシンボリックリンク  |
|  run  |  実行中のステータスを保存するディレクトリ。`/run/httpd` へのシンボリックリンク  |

Apache はメインとなる一つの設定ファイルがあり、そこから追加の設定ファイルを読み込む方式で構成されている。

設定ファイルを見てみる
---

ディレクトリやファイル、パス単位で設定範囲を指定したい場合は、その適用範囲を以下のように `<Hoge target> ~ </Hoge>` と囲んで指定する。

```bash
<Directory "/var/www">
    AllowOverride None
    # Allow open access:
    Require all granted
</Directory>
```

これをディレクティブという。

ディレクティブでモジュールを指定する場合は `Module` ではなく `IfModule` を記述する。これは「もしモジュールがあれば」という意味で、もしモジュールがない場合でもエラーにならず設定ファイルが使えるようになっている。

## 基本的な設定

設定ファイルの場所
---

設定ファイルを置く場所を表す。これ以降の設定ファイル中で `/` で始まらないディレクトリはこのディレクトリを起点とすることを示す。

```bash
ServerRoot "/etc/httpd"
```

受信する（Listen する）ポート番号
---

```bash
Listen 80
```

実行ユーザーとグループ
---

```bash
User apache
Group apache
```

追加の設定ファイルを読み込む①
---

```bash
Include conf.modules.d/*.conf
```

yum で追加したモジュールが使用する設定ファイルを読み込む。モジュールを追加したときに作られる。

追加の設定ファイルを読み込む②
---

```bash
IncludeOptional conf.d/*.conf
```

管理者が追加する設定ファイル。複数ドメイン運用時のそれぞれのドメインに対する設定などはここに書くのが通例。

不具合連絡先のメールアドレス
---

```bash
ServerAdmin root@localhost
```

サーバー名
---

```bash
#ServerName www.example.com:80
```

サーバー名はデフォルトでコメントアウトされて無効になっており、自動判別するようになっている。サーバー本体の名称が外向けのサーバー名と違う名前のときはコメントアウトを外しサーバー名を指定する。

ルート以下すべてのディレクトリに関する設定
---

```bash
<Directory />
    AllowOverride none
    Require all denied
</Directory>
```

ルートディレクトリ以下、すべてのディレクトリに対してまとめて設定している。最初にすべての操作を禁止し、個別に許可する形式になっている。

- `AllowOverride none`

上書き不可。設定の上書きは .htaccess で行うため、実質 .htaccess の無効化のようなもの。

- `Require all denied`

Apache のアクセス制御ディレクティブの一つで、すべてのリクエストを拒否することを指示する指示文。

このディレクティブが使用される場合、Apache は、それが含まれる場所（通常、VirtualHost、Directory、または Location ブロック）に関連付けられたリソースへのすべてのリクエストを拒否する。つまり、誰もリソースにアクセスできず、HTTP ステータスコード 403 Forbidden が返される。

デフォルトのファイルを指定
---

```bash
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

ファイルを指定せずに URL にアクセスされた場合にデフォルトで表示するファイルを指定する。

.ht で始まるファイルのアクセスに関する設定
---

```bash
<Files ".ht*">
    Require all denied
</Files>
```

.ht で始まるファイルは設定情報やパスワードが記載されているのでブラウザから参照されると問題がある。なのでアクセスできないように設定している。

デフォルトの文字コード
---

```bash
AddDefaultCharset UTF-8
```

ドキュメントルートの場所
---

公開領域の親となる `/var/www` の中にドキュメントルート `/var/www/html` と CGI の公開領域が含まれる。それらの設定をする。

```bash
DocumentRoot "/var/www/html"
```

公開領域の親ディレクトリの設定
---

上書き不可、アクセスはすべて許可になっている。つまりウェブでアクセスできることを示す。

```bash
<Directory "/var/www">
    AllowOverride None
    # Allow open access:
    Require all granted
</Directory>
```

ドキュメントルートの設定
---

Options が設定されている。

```bash
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

- Options で設定できる値と意味

|  値  |  意味  |
| ---- | ---- |
|  All  |  MultiViews を除いたすべての機能  |
|  ExecCGI  |  mod_cgi による CGI スクリプトの実行  |
|  FollowSymLinks  |  シンボリックをたどる  |
|  Includes  |  mod_include による SSI 機能  |
|  IncludesNOEXEX  |  mod_include による SSI 機能だが #exec と #exec CGI を使ったプログラムの実行は不可能  |
|  Indexes  |  DirectoryIndex で指定したファイルがない時、格納されているファイル一覧を表示する  |
|  MultiViews  |  コンテンツネゴシエーション機能を有効にする  |
|  SymLinksIfOwmMatch  |  所有者が合致するときだけシンボリックリンクをたどる  |

CGI を実行する場所
---

```bash
<IfModule alias_module>
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
</IfModule>
```

`ScriptAlias` で CGI を実行可能な場所を定義する。`/var/www/cgi-bin/` においたファイルは `/cgi-bin/` というパスでアクセスでき実行可能であることを示す。

CGI ディレクトリの場所
---

```bash
<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>
```

CGI のアクセス権を設定。

その他の設定
---

|  設定内容  |  意味  |
| ---- | ---- |
|  `ErrorLog "logs/error_log"`  |  エラーログを指定した箇所に書き込む  |
|  `LogLevel warn`  |  エラーログに記載するレベルを指定した値に設定する  |
|  `<IfModule log_config_module>` `<IfModule logio_module>`  |  アクセスログを指定した箇所に書き込む  |
|  `<IfModule mime_module>` `<IfModule mime_magic_module>`  |  ファイルの拡張子ごとに MIME タイプと呼ばれるファイルの場所を示す情報をクライアントに送信するための設定  |
|  `#EnableMMAP off` `EnableSendfile on`  |  OS がサポートしている場合、パフォーマンスを向上する設定  |

設定ファイルを書き換えたら再起動する
---

```bash
systemctl restart httpd
```
