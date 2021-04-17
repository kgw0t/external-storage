---
title: Spring BootでREST APIサーバーを構築する
tags:
  - Mac
  - Spring
  - Spring Boot
  - REST API
  - Java
  - MariaDB
categories:
  - tech
comments: false
date: 2021-04-18 01:43:43
---


JavaでREST APIサーバーを立ててみたいと思い、よく利用されているSpring Bootを利用して構築する。

# 実施時の環境
- Mac
  - Big Sur 11.2.3
- Java
  - Amazon Corretto 11
- [MariaDB](/tech/6b07e2135da3/)
  - 10.5.9-MariaDB Homebrew

# Springプロジェクトの作成
[Spring Initializr](https://start.spring.io/)でSpringプロジェクトの雛形を作成できるので、これを利用する。

<br>
{% asset_img 1.png 500 %}

- Project
  - Gradleの方が多少慣れているのでGradleを選択
- Language
  - デフォルトがJavaだったのでそのまま
- Spring Boot
  - デフォルトで安定板の最新っぽいのが選択されているのでそのまま
- Project Metadata
  - Group, Artifact, Descriptionを適当に入力する
    - NameはArtifactと同じものが自動入力される
    - Package nameはGroup, Artifactから自動入力される
  - ローカルで動かすだけなのでPackagingには特に拘らずデフォルトのJar
  - Javaは利用するバージョンに合わせて11を選択

Dependenciesから依存ライブラリを簡単に追加できる。
今回利用するものは以下。

- Spring Data JPA
  - RDBをJavaで操作するためのAPI(JPA : Java Persistence API)を提供するライブラリ
- MariaDB Driver
  - JavaでMariaDBに接続するためのライブラリ
- Spring Web
  - Spring MVCを利用し、Webアプリケーションを構築するためのライブラリ
  - Tomcatをデフォルトの埋め込みコンテナとして利用する
- Lombok
  - Javaの冗長な記述をアノテーションに置き換えることで、簡潔に記述できるようにするライブラリ

設定後は「GENERATE」し、zipファイルをダウンロードする。
ダウンロードしたzipファイルを好きな場所で展開すると、Springプロジェクトの雛形が利用できる。

ただし、この状態でアプリケーションを起動してもエラーで落ちる。
アプリケーションの起動時にデータベースとの接続チェックを行い、接続に失敗した場合はアプリケーションが起動しないようなので、先にデータベースの設定をする必要がある。

# MariaDB
MariaDBを起動するところまでは[こちら](/tech/6b07e2135da3/)。
テスト用のデータとして、今回はブログを想定したデータベース、テーブルを作成する。

```zsh
$ mysql -u root -p

MariaDB [(none)]> create database blog;
MariaDB [(none)]> use blog;
MariaDB [blog]> create table article (
    -> id int primary key auto_increment,
    -> title text not null,
    -> body text not null,
    -> author text not null,
    -> updated_at datetime not null
    -> );
```

これで、`blog`データベースと`article`テーブルの作成が完了した。

次に、このデータベースにアクセスするユーザーを作成する。

```zsh
MariaDB [blog]> create user app@localhost identified by 'password';
MariaDB [blog]> select Host, User from mysql.user where Host = 'localhost' and User = 'app';
+-----------+------+
| Host      | User |
+-----------+------+
| localhost | app  |
+-----------+------+
```

パスワードを`password`にしているが、ちゃんとしたシステムでは使わないこと。
今回はローカルでしか動作させないのでこれでいく。

ユーザーを作っただけだと、下記のように何も権限が設定されていない。

```zsh
MariaDB [blog]> show grants for app@localhost;
+------------------------------------------------------------------------------------------------------------+
| Grants for app@localhost                                                                                   |
+------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `app`@`localhost` IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19' |
+------------------------------------------------------------------------------------------------------------+
```

作成したユーザーに対して、`blog`データベースに対する権限を設定する。

```zsh
MariaDB [blog]> grant all on blog.* to app@localhost;
MariaDB [blog]> show grants for app@localhost;
+------------------------------------------------------------------------------------------------------------+
| Grants for app@localhost                                                                                   |
+------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `app`@`localhost` IDENTIFIED BY PASSWORD '*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19' |
| GRANT ALL PRIVILEGES ON `blog`.* TO `app`@`localhost`                                                      |
+------------------------------------------------------------------------------------------------------------+
```

これでデータベースの準備は完了となる。

## アプリケーションを起動してみる
> アプリケーションの起動時にデータベースとの接続チェックを行い、接続に失敗した場合はアプリケーションが起動しないようなので、先にデータベースの設定をする必要がある。

データベースの準備が完了したため、接続チェックを通すことができる。
作成したSpringプロジェクト配下の設定ファイルにデータベースに関する情報を追記する。

```zsh
$ cat src/main/resources/application.properties
spring.datasource.url=jdbc:mariadb://localhost:3306/blog
spring.datasource.username=app
spring.datasource.password=password
```

追記後にアプリケーションを立ち上げる。

```zsh
$ ./gradlew clean build
$ ./gradlew bootRun
```

`localhost:8080`にアクセスすると、下記のように404が返却されるが、アプリケーション自体は立ち上がっていることがわかる。

<br>
{% asset_img 2.png 500 %}

# REST APIを作成する
`article`テーブルに対するGET,POST,PUT,DELETEの4種類のリクエストを受け取れるエンドポイントを作成する。
それぞれのエンドポイントの仕様はざっくり

- 記事データ取得API
  - GET
  - `{host}/article/{id}`
  - 指定された`id`に対応する記事データを返却する
- 記事データ新規作成API
  - POST
  - `{host}/article`
    - リクエストボディで記事データを受け取る
  - 受け取った記事データをDBに格納し、格納した記事データをそのまま返却する
- 記事データ更新API
  - PUT
  - `{host}/article/{id}`
    - リクエストボディで記事データを受け取る
  - 指定された`id`に対応する記事データを、受け取った記事データで更新し、更新した記事データををのまま返却する
- 記事データ削除API
  - DELETE
  - `{host}/article/{id}`
  - 指定された`id`に対応する記事データを削除する

という感じで作成する。
`{host}`は今回だと`localhost:8080`になる。

REST APIを構築するためには

- モデルクラスの作成
- リポジトリインターフェースの作成
- コントローラクラスの作成
  - 記事データ取得APIの作成
  - 記事データ新規作成APIの作成
  - 記事データ更新APIの作成
  - 記事データ削除APIの作成

という作業が必要である。

## モデルクラスの作成
`article`テーブルにアクセスするためのモデルクラスを下記のように作成した。

```java
package work.rest.api.bean;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.io.Serializable;
import java.time.LocalDateTime;

@Getter
@Setter
@Entity
@Table(name = "article")
public class Article implements Serializable {
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String title;

    private String body;

    private String author;

    @Column(name = "updated_at")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime updatedAt;
}
```

getter, setterはlombokの`@Getter, @Setter`アノテーションで自動生成するため、記述は不要となる。

モデルクラスには`@Entity`アノテーションを付与し、`@Table`アノテーションでテーブル名を指定すること。

一般的に`Serializable`を実装するみたいなのでその通りにしているが正直細かくなぜそうするのかは理解できていない。
パッと思いつくのはキャッシュすることを考えると、`Serializable`である必要があるというくらい。

`id`カラムはテーブル作成時に`auto_increment`としているため、`@GenerateValue`アノテーションでテーブルのidentity列を利用するように設定しておく。

`updated_at`カラムはアンダースコアが入るが、Javaの規約上変数名をキャメルケースにしたいので、`@Column`アノテーションで対応するカラム名を指定する。
また、フォーマットに関しても想定しているフォーマットを`@JsonFormat`アノテーションで指定している。

## リポジトリインターフェースの作成
モデルクラスとデータベースの中継をするリポジトリインターフェースを下記のように作成した。

```java
package work.rest.api.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import work.rest.api.bean.Article;

public interface ArticleRepository extends JpaRepository<Article, Integer> {
}
```

`JpaRepository`クラスを継承し、ジェネリクスは対象のモデルクラスと、モデルクラスで`@id`アノテーションを付与した変数のクラス（今回はプリミティブ型に対するラッパークラス）を指定すること。

## コントローラクラスの作成
各APIのエンドポイントとなるコントローラクラスを以下のように定義した。

```java
package work.rest.api.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import work.rest.api.bean.Article;
import work.rest.api.repository.ArticleRepository;

import java.time.LocalDateTime;
import java.util.Optional;

@RestController
public class ArticleController {
    @Autowired
    ArticleRepository articleRepository;
}
```

API用のコントローラなので、`@RestController`アノテーションをクラスに付与している。

コントローラクラス単位でリクエストマッピングの設定もできるが、今回は個別のメソッドごとに設定した。

`article`テーブルの操作となるので、対応するリポジトリクラスを定義し、`@Autowired`アノテーションを付与することでDIする。

### 記事データ取得APIの作成

```java
    @RequestMapping(value = "/article/{id}", method = RequestMethod.GET)
    public Article get(@PathVariable int id) {
        Optional<Article> target = articleRepository.findById(id);
        return target.orElse(null);
    }
```

`@RequestMapping`アノテーションで、エンドポイントとメソッドを指定する。

パスパラメータを受け取るために引数の`id`に`@PathVariable`を指定する。

後はリポジトリクラスを利用してレコードを引くだけ。
Optionalが返却されるので、ない場合は`null`を指定している。

### 記事データ新規作成APIの作成

```java
    @RequestMapping(value = "/article", method = RequestMethod.POST)
    public Article post(@RequestBody Article article) {
        LocalDateTime now = LocalDateTime.now();
        article.setUpdatedAt(now);

        Article newArticle = articleRepository.save(article);
        return newArticle;
    }
```

引数はArticleクラスと同等のデータをリクエストボディから受け取るために`@RequestBody`アノテーションを付与する。

`updated_at`にLocalDateTimeクラスで現在時刻を設定し、リポジトリクラスを利用して新規レコード追加する。

リポジトリクラスからは新規追加されたデータが返ってくるのでそのままレスポンスとして返却する。

### 記事データ更新APIの作成

```java
    @RequestMapping(value = "/article/{id}", method = RequestMethod.PUT)
    public Article put(@PathVariable int id, @RequestBody Article article) {
        Optional<Article> target = articleRepository.findById(id);
        target.ifPresent(t -> {
            LocalDateTime now = LocalDateTime.now();
            t.setTitle(article.getTitle());
            t.setBody(article.getBody());
            t.setAuthor(article.getAuthor());
            t.setUpdatedAt(now);

            articleRepository.save(t);
        });
        return target.orElse(null);
    }
```

特筆することはなし。

### 記事データ削除APIの作成

```java
    @RequestMapping(value = "/article/{id}", method = RequestMethod.DELETE)
    public Article delete(@PathVariable int id) {
        Optional<Article> target = articleRepository.findById(id);
        target.ifPresent(t -> {
            LocalDateTime now = LocalDateTime.now();
            t.setUpdatedAt(now);

            articleRepository.delete(t);
        });
        return target.orElse(null);
    }
```

こちらも特筆することなし。

# Rest APIを動かしてみる
起動コマンドは先ほどと同様。

```zsh
$ ./gradlew clean build
$ ./gradlew bootRun
```

## 記事データの新規作成
初期状態では`article`テーブルにレコードが存在しないため、まずは記事データ新規作成APIを利用してデータを作成する。
記事データ新規作成APIに送信するリクエストボディは以下。

```json
# post.json
{
    "title": "Spring BootでREST　APIサーバーを立てる",
    "body": "環境 : MacOS, Sprint Boot, MariaDB .....",
    "author": "kgw0t"
}
```

最低限外から渡す必要がある`title`, `body`, `author`をJSON形式で送る。

curlコマンドを用いて、記事データ新規作成APIを叩く。

```zsh
$ curl -H "Content-Type: application/json" -XPOST 'localhost:8080/article' -d@post.json
{"id":1,"title":"Spring BootでREST　APIサーバーを立てる","body":"環境 : MacOS, Sprint Boot, MariaDB .....","author":"kgw0t","updatedAt":"2021-04-17 00:57:35"}
```

レスポンスは登録された記事データが返却されている。

```zsh
$ mysql -u root -p blog

MariaDB [blog]> select * from article;
+----+--------------------------------------------------+--------------------------------------------+--------+---------------------+
| id | title                                            | body                                       | author | updated_at          |
+----+--------------------------------------------------+--------------------------------------------+--------+---------------------+
|  1 | Spring BootでREST　APIサーバーを立てる           | 環境 : MacOS, Sprint Boot, MariaDB .....   | kgw0t  | 2021-04-17 00:57:35 |
+----+--------------------------------------------------+--------------------------------------------+--------+---------------------+
```

実際にDBのデータを表示しても同様のデータが存在することが確認できる。

## 記事データの取得
記事データ取得APIを利用し、DBに格納されている記事データを取得する。

```zsh
$ curl -XGET 'localhost:8080/article/1'
{"id":1,"title":"Spring BootでREST　APIサーバーを立てる","body":"環境 : MacOS, Sprint Boot, MariaDB .....","author":"kgw0t","updatedAt":"2021-04-17 00:57:35"}
```

指定する`id`パラメータは記事データ新規作成APIの返却値に含まれていた`id:1`を指定することで、DBのデータと全く同様のものが返却されていることがわかる。

```zsh
$ curl -XGET 'localhost:8080/article/2'
```

なおこのように存在しない`id`パラメータを指定しても、`null`を返却するようにしているため何も返ってこない。

## 記事データの更新
記事データ更新APIを利用し、DBに格納されている記事データを書き換える。
記事データ更新APIに送信するリクエストボディは以下。

```json
# put.json
{
    "title": "更新された",
    "body": "更新された",
    "author": "更新された"
}
```

全てのデータが`更新された`になり、`updated_at`の値がリクエストを送信した時間になっていれば問題なしというデータである。

```zsh
$ curl -H "Content-Type: application/json" -XPUT 'localhost:8080/article/1' -d@put.json
{"id":1,"title":"更新された","body":"更新された","author":"更新された","updatedAt":"2021-04-18 00:00:35"}
```

レスポンスは更新後の記事データなので、全て想定通りに動作していることがわかる。

```zsh
$ mysql -u root -p blog

MariaDB [blog]> select * from article;
+----+-----------------+-----------------+-----------------+---------------------+
| id | title           | body            | author          | updated_at          |
+----+-----------------+-----------------+-----------------+---------------------+
|  1 | 更新された      | 更新された      | 更新された      | 2021-04-18 00:00:35 |
+----+-----------------+-----------------+-----------------+---------------------+
```

DBを見ても正しく更新されていることが確認できる。
また、GETリクエストと同じ場所にアクセスしているが、メソッドの違いで動作が異なっていることがわかる。

## 記事データの消去
記事データ削除APIを利用し、DBに格納されている記事データを消去する。

```zsh
$ curl -XDELETE 'localhost:8080/article/1'
{"id":1,"title":"更新された","body":"更新された","author":"更新された","updatedAt":"2021-04-18 00:21:43"}
```

レスポンスとしては削除対象となったデータが返却されている。

```zsh
$ mysql -u root -p blog

MariaDB [blog]> select * from article;
Empty set (0.000 sec)
```

DBを確認すると削除されていることが確認できる。

# おわりに
Spring Bootを利用したREST APIを作成し、動作確認までできた。
実際にシステムとして稼働させる場合はパラメータのバリデーションや、データが存在しない場合のレスポンス等、まだまだ処理が足りていない部分があることを忘れないようにしたい。

コードは[こちら](https://github.com/kgw0t/spring-boot-rest-api)。