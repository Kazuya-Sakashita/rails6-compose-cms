# RAILS.md

[oiax/rails6-compose](https://github.com/oiax/rails6-compose) をベースに Rails 6 アプリケーションの開発・学習を始める手順を解説します。

## アプリケーション開発の方針

* データベース管理システム（DBMS）には PostgreSQL を使用する。
* Sprockets を使わない。
* Webpacker を使う。

## 凡例

* 基本的に、ターミナルでコマンドを実行することで作業が進んでいきます。
* コマンドを入力する際には、行頭にある `%` 記号および `$` 記号を省いてください。
* `%` 記号の付いたコマンドは、ホスト OS のターミナル（Windows の場合は、コマンドプロンプト）で入力してください。
* `$` 記号の付いたコマンドは、Web コンテナのターミナルで入力してください。

## Railsアプリケーションの骨格を作る

Webコンテナにログインして以下のコマンド群を実行します。

```
$ cd /apps
$ rails new myapp -BJS -d postgresql
```

※ `myapp` の部分は適宜置き換えてください。ここに指定された文字列がアプリケーション名となります。

### 指定されたオプションの説明

* `-B`: `bundle install` の実行を後回しにする。
* `-J`: JavaScriptファイル群を生成しない。
* `-S`: Sprocketsファイル群を生成しない。
* `-d`: 選択されたデータベース管理システム（PostgreSQL）用の設定ファイルを生成する

※ Webpacker は後で導入します。

## Webコンテナからログアウト

```
$ exit
```

## ソースコードの編集

* ホスト OS 側のテキストエディタで Rails アプリケーションのソースコードを編集します。
* ソースコードは `rails6-compose/apps/myapp` フォルダにあります。

### `Gemfile` の編集

テキストエディタで `Gemfile` を開き、次のような記述を見つけてください。

```
gem 'bootsnap', '>= 1.1.0', require: false
```

この直後に、次の記述を挿入してください。

```
gem 'webpacker'
```

`Gemfile` 末尾にある次の記述を削除してください。

```
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

### `config/database.yml` の編集

テキストエディタで `config` ディレクトリにある `database.yml` を開き、次のような記述を見つけてください。

```
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
```

この直後に、次の記述を挿入してください。

```
  host: db
  username: postgres
  password:
```

※ 各行の先頭に半角スペースを2個置いてください。

## Rails アプリケーションのセットアップ

Web コンテナにログインして以下のコマンド群を実行します。

```
$ cd /apps/myapp
$ bundle
$ bin/rails webpacker:install
$ bin/rails db:create
```

※ `myapp` の部分は適宜置き換えてください。

## スタイルシートの移設

引き続き、Web コンテナで以下のコマンド群を実行します。

```
$ mkdir app/javascript/stylesheets
$ cp app/assets/stylesheets/scaffold.css app/javascript/stylesheets
$ rm -rf app/assets
```

テキストエディタで `app/javascript/packs/application.js` を開き、内容をすべて削除してから、次の内容を書き加えます。

```
import "../stylesheets/scaffold.css";
```

## レイアウトテンプレートの書き換え

テキストエディタで `app/javascript/packs/application.js` を開き、次のような記述を見つけてください。

```
    <%= stylesheet_link_tag    'application', media: 'all' %>
```

この部分を次のように書き換えてください。

```
    <%= stylesheet_pack_tag    'application', media: 'all' %>
```

## Scaffold の作成

Web コンテナで以下のコマンド群を実行します。

```
$ bin/rails g scaffold user name:string
$ bin/rails db:migrate
```

2番目のコマンドを実行した結果、ターミナルに次のように表示されればOKです。ただし、数字の部分は異なります。

```
== 20190219014032 CreateUsers: migrating ======================================
-- create_table(:users)
   -> 0.0092s
== 20190219014032 CreateUsers: migrated (0.0093s) =============================
```

## Rails アプリケーションの起動

ホストOSで別のターミナルを開き、次のコマンドを実行します。

```
% docker-compose exec web bash -c 'cd /apps/myapp; bin/webpack-dev-server'
```

さらに別のターミナルを開き、次のコマンドを実行します。

```
% docker-compose exec web bash -c 'cd /apps/myapp; bin/rails s'
```

## ブラウザで動作確認

ホスト OS 側のブラウザで http://localhost:3000/users を開き、「Users」というタイトルのページが開くことを確認します。そして、ユーザーの追加、編集、削除ができることを確認します。

## Rails アプリケーションの停止

Rails アプリケーション起動のために開いたふたつのターミナルでそれぞれ `Ctrl-C` キーを押すと、Rails アプリケーションが停止します。

## コンテナ群の停止

開発作業を終えてパソコンの電源を切る前に Db コンテナと Web コンテナを停止します。

```
% docker-compose stop
```

ソースコードやデータベースの中身は保存されているので、ホスト OS を起動しなおしても `docker-compose up -d` コマンドでコンテナ群を起動すれば、前回の状態から開発作業を続行できます。