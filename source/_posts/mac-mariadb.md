---
title: Mac上でMariaDBを動かす
tags:
  - mac
  - mariadb
categories:
  - tech
comments: false
date: 2021-03-25 01:13:30
---


MySQLは使ったことあったけど、MariaDBってどんな感じなのかなと思い、とりあえずMac上で動かしてみる。

# 実施時の環境
- Mac
  - Big Sur 11.2.3
- [Homebrew](https://brew.sh/index_ja)
  - macOS用のパッケージマネージャ
  - MariaDBをインストールするために利用

# [MariaDB](https://mariadb.org/)
Homebrewを利用してMariaDBをインストールする。

```zsh
$ brew install mariadb
```

## MariaDBを立ち上げる
下記のコマンドでMariaDBが立ち上がる。

```zsh
brew services start mariadb
```

止めたい時は

```zsh
brew services stop mariadb
```

で止まる。

## 初期設定
立ち上げ後はまず初期設定を行うこと。

```zsh
$ sudo mysql_secure_installation
Password:

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] Y ...①
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] Y ...②
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y ...③
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y ...④
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y ...⑤
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y ...⑥
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

所々選択する必要があるが、それぞれの意味は下記にまとめる。

① osにログインしているのユーザー名とDBのユーザー名が同じであれば、UnixSocketを使って認証するか？ [Y/n]
特に利用しないので、「Y」を入力し、認証しない。

② ルートユーザーのパスワードを変更するか？ [Y/n]
初期状態だと設定無しになるので、「Y」を入力し、設定しておく。

③ 無名ユーザーを削除するか？ [Y/n]
誰でも接続できてしまうので、「Y」を入力し、無名ユーザーを削除する。

④ ルートユーザーでのリモート接続を拒否するか？ [Y/n]
リモートで接続することはないので、「Y」を入力し、接続できないようにしておく。

⑤ テストデータベースを削除するか？ [Y/n]
特に不要なので、「Y」を入力し、削除する。

⑥ 上記の変更を即座に反映するか？ [Y/n]
反映させたいので、「Y」を入力し、反映させる。

## MariaDBに接続する

```zsh
$ sudo mysql -u root -p
```

初期設定の際に決めたrootユーザーのパスワードを入力すれば接続できる。

