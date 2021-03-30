---
title: nodenvを利用してMacにNode.jsを入れる&バージョン管理
tags:
  - Mac
  - nodenv
  - Node.js
categories:
  - tech
date: 2021-03-11 22:46:14
comments: false
---


Node.jsが必要になったので、Macにインストールする。
せっかくなので、nodenvを利用してNode.jsのバージョン管理をできるようにする。

# 実施時の環境
- Mac
  - Big Sur 11.2.3
- [Homebrew](https://brew.sh/index_ja)
  - macOS用のパッケージマネージャ
  - nodenvをインストールするために利用

# [nodenv](https://github.com/nodenv/nodenv)
Homebrewを利用してnodenvをインストールする。

```zsh
$ brew install nodenv
```

## 初期設定
インストール完了後は `~/.zshrc` に `eval "$(nodenv init -)"` を記述する。

```zsh
$ vi ~/.zshrc
$ cat ~/.zshrc
...(略)
# nodenv
eval "$(nodenv init -)"

$ source ~/.zshrc
```

何を記述すればいいかわからなくなった場合は、 `nodenv init` を実行すればわかる。
どのファイルに記載すればいいかも教えてくれるのが地味に嬉しい。

```zsh
$ nodenv init
# Load nodenv automatically by appending
# the following to ~/.zshrc:

eval "$(nodenv init -)"
```

# Node.js
インストールしたnodenvを利用して、Node.jsをインストールする。
まずはどのバージョンをインストールできるかを確認する。

```zsh
$ nodenv install -l
...(略)

15.8.0
15.9.0
15.10.0
15.11.0
chakracore-dev
chakracore-nightly
chakracore-8.1.2
chakracore-8.1.4

...(略)
```

とりあえず現時点で純粋なNode.jsで最新っぽい `15.11.0` をインストールし、Mac全体で利用できるように設定する。

```zsh
$ nodenv install 15.11.0

# Mac全体で 15.11.0 を利用するように設定
$ nodenv global 15.11.0
```

## 個別のプロジェクトへの適用
とりあえず何も設定していない場所でも使えるようにグローバルに設定をしたが、グローバルのバージョンが変わった時にそれぞれのプロジェクトで別バージョンを見に行かないようにプロジェクトごとに設定しておく。

```zsh
$ cd {プロジェクトへのパス}
$ nodenv local 15.11.0
```

## インストールされているかの確認
指定したNode.jsのバージョンがインストールされているかを確認する。

```zsh
$ node -v
v15.11.0
```

これでOK。
あとはnodenvでインストールしたNode.jsのバージョン一覧を確認してみる。

```zsh
$ nodenv versions
  system
* 15.11.0 (set by /Users/xxx/yyy)
```

こんな感じ。
今後色々なバージョンが増えてくると結果がいっぱい返ってくるので何をインストールしているかを確認できる。
