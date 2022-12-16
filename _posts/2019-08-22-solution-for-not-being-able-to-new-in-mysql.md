---
layout: post
title: Ruby on Rails でデータベースを mysql で new できない場合の解決法
date: 2019/08/22
author: otomi
tags: [blog]
comments: false
---

データベースに `mysql` 使いたいなぁ！と思って

    $ rails new i\_want\_to\_use\_mysql -d mysql 

したら、エラーで先に進まない場合の解決法。

<!-- more -->

    $ rails new i\_want\_to\_use\_mysql -d mysql （中略） Installing mysql2 0.5.2 with native extensions Gem::Ext::BuildError: ERROR: Failed to build gem native extension. current directory: /Users/aksk/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/mysql2-0.5.2/ext/mysql2 /Users/aksk/.rbenv/versions/2.6.3/bin/ruby -I /Users/aksk/.rbenv/versions/2.6.3/lib/ruby/2.6.0 -r ./siteconf20190822-76368-1osl1a0.rb extconf.rb checking for rb\_absint\_size()... yes checking for rb\_absint\_singlebit\_p()... yes checking for rb\_wait\_for\_single\_fd()... yes checking for -lmysqlclient... no ----- mysql client is missing. You may need to 'brew install mysql' or 'port install mysql', and try again. ----- \*\*\* extconf.rb failed \*\*\* Could not create Makefile due to some reason, probably lack of necessary libraries and/or headers. Check the mkmf.log file for more details. You may need configuration options. Provided configuration options: --with-opt-dir --without-opt-dir --with-opt-include --without-opt-include=${opt-dir}/include --with-opt-lib --without-opt-lib=${opt-dir}/lib --with-make-prog --without-make-prog --srcdir=. --curdir --ruby=/Users/aksk/.rbenv/versions/2.6.3/bin/$(RUBY\_BASE\_NAME) --with-mysql-dir --without-mysql-dir --with-mysql-include --without-mysql-include=${mysql-dir}/include --with-mysql-lib --without-mysql-lib=${mysql-dir}/lib --with-mysql-config --without-mysql-config --with-mysql-dir --without-mysql-dir --with-mysql-include --without-mysql-include=${mysql-dir}/include --with-mysql-lib --without-mysql-lib=${mysql-dir}/lib --with-mysqlclientlib --without-mysqlclientlib To see why this extension failed to compile, please check the mkmf.log which can be found here: /Users/aksk/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/extensions/x86\_64-darwin-18/2.6.0-static/mysql2-0.5.2/mkmf.log extconf failed, exit code 1 Gem files will remain installed in /Users/aksk/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/mysql2-0.5.2 for inspection. Results logged to /Users/aksk/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/extensions/x86\_64-darwin-18/2.6.0-static/mysql2-0.5.2/gem\_make.out An error occurred while installing mysql2 (0.5.2), and Bundler cannot continue. Make sure that `gem install mysql2 -v '0.5.2' --source 'https://rubygems.org/'` succeeds before bundling. In Gemfile: mysql2 run bundle exec spring binstub --all Could not find gem 'mysql2 (\>= 0.4.4, \< 0.6.0)' in any of the gem sources listed in your Gemfile. Run `bundle install` to install missing gems.

ってエラーが出て自分では全く解決できる気がしなかったので解決方法をメモっておく。

自分では全く解決できる気がしなかった理由は、エラーメッセージ内に、このコマンド打ってねって記載されているにも関わらず、その通りにしても解決できなかったから。

以下の記事をそのまま参考にさせてもらってます。

<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fqiita.com%2Ffukudakumi%2Fitems%2F463a39406ce713396403" title="【Rails】MySQL2がbundle installできない時の対応方法 - Qiita" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"></iframe><cite class="hatena-citation"><a href="https://qiita.com/fukudakumi/items/463a39406ce713396403">qiita.com</a></cite>

    $ brew info openssl

して、表示されるメッセージ内の

    openssl: stable 1.0.2s (bottled) [keg-only] SSL/TLS cryptography library https://openssl.org/ /usr/local/Cellar/openssl/1.0.2s (1,795 files, 12.0MB) Poured from bottle on 2019-07-25 at 18:36:37 From: https://github.com/Homebrew/homebrew-core/blob/master/Formula/openssl.rb ==\> Caveats A CA file has been bootstrapped using certificates from the SystemRoots keychain. To add additional certificates (e.g. the certificates added in the System keychain), place .pem files in /usr/local/etc/openssl/certs and run /usr/local/opt/openssl/bin/c\_rehash openssl is keg-only, which means it was not symlinked into /usr/local, because Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries. If you need to have openssl first in your PATH run: echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' \>\> ~/.zshrc For compilers to find openssl you may need to set: export LDFLAGS="-L/usr/local/opt/openssl/lib" export CPPFLAGS="-I/usr/local/opt/openssl/include" For pkg-config to find openssl you may need to set: export PKG\_CONFIG\_PATH="/usr/local/opt/openssl/lib/pkgconfig" ==\> Analytics install: 502,127 (30 days), 1,741,224 (90 days), 6,608,463 (365 days) install\_on\_request: 60,851 (30 days), 232,924 (90 days), 888,787 (365 days) build\_error: 0 (30 days)

     export LDFLAGS="-L/usr/local/opt/openssl/lib" export CPPFLAGS="-I/usr/local/opt/openssl/include"

この部分の-Lと-Iから始まる部分をコピー。

    $ bundle config --local build.mysql2 "--with-cppflags=-I/usr/local/opt/openssl/include" $ bundle config --local build.mysql2 "--with-ldflags=-L/usr/local/opt/openssl/lib"

上記の２コマンドを実行したあと、

    $ rails new i\_want\_to\_use\_mysql -d mysql 

でnewできました！

なぜこのエラーが発生して、なぜこの方法で解決できるのかが全く理解できていないので、ちょっとずつ上記記事のリンク先などを読んでいきたい。
