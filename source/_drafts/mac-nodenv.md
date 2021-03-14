---
title: Nodenvを利用してMacにNode.jsを入れる&バージョン管理
categories:
- tech
tags: 
- mac
- nodenv
- nodejs
---

Node.jsが必要になったので、Macにインストールしようと思う。
せっかくなので、Nodenvを利用してNode.jsのバージョン管理をできるようにする。

## 実施時の環境
- Mac
  -  OS: Big Sur 11.2.3
  -  [Homebrew](https://brew.sh/index_ja)
     -  macOS用のパッケージマネージャ
     -  Nodenvをインストールするために利用

## [Nodenv](https://github.com/nodenv/nodenv)

### インストール
まずは、Homebrewを利用してNodenvをインストールする。

```zsh
$ brew install nodenv
```

### 初期設定
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

## Node.js
インストールしたNodenvを利用して、Node.jsをインストールする。

### インストール
インストール前にどのバージョンをインストールできるかを確認する。

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

### 個別のプロジェクトへの適用
とりあえず何も設定していない場所でも使えるようにグローバルに設定をしたが、グローバルのバージョンが変わった時にそれぞれのプロジェクトで別バージョンを見に行かないようにプロジェクトごとに設定しておく。

```zsh
$ cd {プロジェクトへのパス}
$ nodenv local 15.11.0
```

### インストールされているかの確認
指定したNode.jsのバージョンがインストールされているかを確認する。

```zsh
$ node -v
v15.11.0
```

これでOK。
あとはNodenvでインストールしたNode.jsのバージョン一覧を確認してみる。

```zsh
$ nodenv versions
  system
* 15.11.0 (set by /Users/xxx/yyy)
```

こんな感じ。
今後色々なバージョンが増えてくると結果がいっぱい返ってくるので何をインストールしているかを確認できる。
