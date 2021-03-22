---
title: Hexoでブログを構築する
tags:
  - mac
  - hexo
categories:
  - tech
date: 2021-03-21 02:51:03
---


ただ書き溜めていくだけの場所が欲しいなと感じ、ブログを構築することにした。
構築する上での欲しい要件は以下である。

- タグがつけられる
- 構築したブログ内のキーワード検索ができる
  - タグを指定した上でのキーワード検索もできる

ブログの構築にあたって、静的サイトジェネレータを使うという点まで決定していた。
タグ付けに関しては基本的にどれを使っても実現できそうだが、難点は検索の方みたい。

そもそも静的サイトジェネレータにおける検索ってなんだ？
検索ページはどうしても動的になるのでは？
と思ったので、もしかしたら自分で作らないととかになるかと考えたこともあった。
ブログの検索はユーザーは使っている割合が少なく、作る必要なんてないという記事も目に入った。

そんな中目に入ったのが、Hexoだった。
[hexo-generator-search](https://github.com/wzpan/hexo-generator-search)というプラグインがあり、簡単に検索機能つけられそうだなと感じた。

現在検索機能まではつけられていないが、一旦Hexoでブログを構築するところまでを書き溜めておく。

# 実施時の環境
- Mac
  - Big Sur 11.2.3
- [Node.js](/tech/e0ba8e9a5528/)
  - 15.11.0

# [Hexo](https://hexo.io/)
[ドキュメント](https://hexo.io/docs/)がわかりやすく、それに沿っていけば問題ない。

## Hexoをインストール

```zsh
$ npm install hexo-cli -g

added 66 packages, and audited 67 packages in 2s

11 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

これだけでHexoコマンドが使えるようになる。

## Hexoプロジェクトの生成
プロジェクトという言葉があっているか微妙だが、作業するディレクトリを作成し、その中で必要なものをインストールする。

```zsh
$ hexo init blog
INFO  Cloning hexo-starter https://github.com/hexojs/hexo-starter.git
INFO  Install dependencies
INFO  Start blogging with Hexo!

$ cd blog
$ npm install

added 6 packages, and audited 187 packages in 1s

15 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

`blog` の部分は任意で好きな名前にすることができる。

## Hello World
とりあえず初期状態で立ち上げて確認する。

```zsh
$ hexo server
or
$ hexo s
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.

$ open http://localhost:4000
```

こんな感じの画面が出てきたら問題なし。
<br>
{% asset_img 1.png 500 %}

# Hexoの設定を変更する
[設定周りの公式ドキュメント](https://hexo.io/docs/configuration)を見ながら少し設定をいじって見る。

## サイト設定を変更してみる
タイトル等基本的な部分の設定をする。

```yml
# Site
title: 外部記憶装置
author: kgw0t
language: ja
timezone: 'Asia/Tokyo'
```

変更後はこんな感じになった。
タイトル部分が変更され、右サイドカラムの部分が日本語になっていることが確認できる。
画像には載っていないが、フッター部に記述されている筆者の名前が変わっている。
タイムゾーンに関しては確認できなかったが、ファイルを作成した日付等に影響するのだろう。
<br>
{% asset_img 2.png 500 %}

## URLの設定をしてみる
URLを自分なりにカスタムできるよう。
デフォルトだと各記事へのリンクが日付やファイル名になっている。
日付情報もURLには特に不要だし、なんとなくファイル名が入ってしまうのはダサいと感じたのでこの辺りを修正してみる。

```yml
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
permalink: :category/:hash/
```

設定したカテゴリとファイル名をハッシュ化したものをパスに設定した。
なお`permalink`の末尾の`/`を削除すると、記事のリンクを謳歌した際に記事がダウンロードされるようになってしまうので消さないようにする。
色々な変数値を当てられるので、[`permalink`の公式ドキュメント](https://hexo.io/docs/permalinks)を見て好きなようにカスタムすると良い。

## テーマの変更をしてみる
[テーマページ](https://hexo.io/themes/)で適用したいテーマを探す。
個人でテーマを作成し、公開している方もいるようなので、このページのテーマにしっくりくるものがなかったらそちらを探してもいいと思う。

個人的にはシンプルな見た目が好きなので、[hexo-theme-light](https://github.com/hexojs/hexo-theme-light)を適用する。

githubのREADMEの通りに設定していく。
まずはリポジトリのクローンを行う。

```zsh
$ git clone --depth 1 https://github.com/hexojs/hexo-theme-light themes/light
Cloning into 'themes/light'...
remote: Enumerating objects: 89, done.
remote: Counting objects: 100% (89/89), done.
remote: Compressing objects: 100% (79/79), done.
remote: Total 89 (delta 0), reused 55 (delta 0), pack-reused 0
Unpacking objects: 100% (89/89), done.
```

次に、テーマの設定を行う。
この時の名前は`themes`ディレクトリ配下のディレクトリ名なので、そこと合わせればクローンする時にどんな名前にしても問題ない。

```yml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: light
```

設定後はこんな感じになる。
自分でテーマを作成することもできるので、そこもできたらなと思うが、今はとりあえずここまで。
<br>
{% asset_img 3.png 500 %}

なおこのテーマを使った場合、ヘッダー部の`Home`リンクが`localhost:4000/null`となってしまう。
テーマ変える前では発生しないため、テーマ側の問題かと思う。
修正のプルリクエスト出してみるか、とりあえずForkして自分用にだけでも直したいと思う。

# 記事を書いてみる
[記事を書く公式ドキュメント](https://hexo.io/docs/writing)を見ながら記事を書いてみる。

## 記事を新規追加する

```zsh
$ hexo new new-article
or
$ hexo n new-article
INFO  Validating config
INFO  Created: ~/blog/source/_posts/new-article.md
```

これでMarkdownのファイルが新規作成できる。
また、どのレイアウトで生成するかを指定することができ、それぞれ下記のような認識である。

- `post(default)` : 公開記事。`source/_posts`配下に作成される。
- `draft` : 非公開記事。`source/_drafts`配下に作成される。
- `page` : 固定ページ。`source`配下に作成される。

```zsh
$ hexo new draft draft-test
INFO  Validating config
INFO  Created: ~/blog/source/_drafts/draft-test.md
```

ドキュメント的にはレイアウトの中には入っていないようだが、`photo`を指定することで、`post`とは別のレイアウトが変化する。
```zsh
$ hexo new photo photo-test
```
<br>
{% asset_img 4.png 500 %}

`draft`で作成した非公開記事は普通にHexoを立ち上げても表示されないが、オプションを指定することで表示される。

```zsh
$ hexo server --draft
```

また、非公開記事を公開する場合は、下記のコマンドで実施できる。
後に記述する記事の設定の日時部分が勝手に入ってくれるので、`draft`で書き始め、`publish`するというのが基本の流れかと思う。

```zsh
$ hexo publish {非公開記事の名前}
or
$ hexo p {非公開記事の名前}
```

公開記事を非公開にする場合は、コマンドが見つからなかった。
とりあえず手動でファイルを非公開記事のディレクトリ配下に移動させればいいので、問題はない。

## 記事の設定を変更する

Markdownのファイルの先頭に記事に関する設定を記述することができる。

```md
---
title: Hexoでブログを構築する
tags:
  - mac
  - hexo
categories:
  - tech
date: 2021-03-21 00:18:03
---
```

タグやカテゴリをつけることで、アクセスがよくなる。
特にカテゴリは、`permalink`の設定で記事のパスに含めるようにしたので、全体でカテゴリのつけ方を統一することでURLも綺麗になる。

## 記事内に画像を表示する
外部に落ちている画像を指定する場合は、Markdownの画像の入れ方に準拠すれば問題なく表示できる。
ここで説明するのは自分で画像を配置し、それを記事内に表示する方法である。

まずは、画像を置くディレクトリを作るために、`_config.yml`の設定を変更する。

```yml
# Writing
post_asset_folder: true
```

`post_asset_folder`を`true`にすることで、新規の記事を追加した際にMarkdownのファイルと共にディレクトリが追加されるようになる。

```zsh
$ hexo new post image-test
$ ls source/_posts/
image-test     image-test.md
```

記事名と同名のフォルダに画像を配置した状態で、記事側に`asset_img`を入れることで、配置した画像を表示できる。
レイアウトが`draft`の場合も、`source/_drafts/`配下にフォルダが作成され、`hexo publish`を実行時も勝手に`source/_posts/`配下にフォルダごと移動してくれる。
ただ、`hexo server --draft`で起動しても画像が表示されなかったため、「`draft`で書き始め、`publish`するというのが基本の流れかと思う。」と記載したが、画像を表示する場合は`post`で書き、確認するしかない。

```zsh
$ ls source/_posts/image-test
1.png
```

```md
{% asset_img 1.png %}
```
<br>
{% asset_img 5.png 500 %}

また、`asset_link`にすることで、画像ファイルへのリンクにすることもできる。

```md
{% asset_link 1.png %}
```
<br>
{% asset_img 6.png 500 %}

画像は`{記事へのURL}/{画像ファイル}`というURLでアクセスすることができる。
もちろんこのURLが把握できていれば、`asset_img`を利用せずにMarkdown形式で画像を表示することもできる。
`asset_img`を利用するメリットは、`permalink`の設定を変更した場合に全ての画像へアクセスするURLが変更されてしまい、画像URLを変更する必要が出てきてしまうため、Markdown形式へのこだわりが無ければこっちを使うと良い。

# 最後に
とりあえず現状やったことベースで記述した。
公式ドキュメントを見るとまだまだ触ったことのない設定等があるので、後々その辺りもまとめられるといい。

`asset_img`では、`permalink`の変更の影響を受けずに画像を設定できるで、考えることが減ったのがよかった。
同ブログ内へのリンクも同様にファイル名の指定のみで実現できないかを調べてみたが、特に情報がなかった。
何か方法があれば是非教えていただきたい。

作成したブログのソースは[こちら](https://github.com/kgw0t/external-storage)。

## 余談
Hexoコマンドのインストール方法として、Advanced installation and usageに記載があるように `hexo-cli` ではなく `hexo` を入れる方法もある。
`hexo-cli` よりも `hexo` の方が更新されているみたいなので、より新しいバージョンを入れることができるかと思って試してみたが、バージョン特に変わっているように見えなかったので、 `hexo-cli` の方で特段問題ないと思う。






























