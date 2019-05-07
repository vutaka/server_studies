# PostgreSQL練習

## ここでやること
* wgetコマンドのインストール
* PostgreSQLのインストール
* PostgreSQLの初期化
* PostgreSQLへの認証方式変更
* PostgreSQLの自動起動の設定
* PostgreSQLのサービス開始
* databaseの作成
* ユーザの作成
* スキーマの作成とpublicスキーマの削除
* DDL実行
* ユーザへテーブルの権限を付与する

## やってみる

### 0.環境構築〜ログインする
* 仮想マシンを立ち上げるためカレントディレクトリが `web_app` であることを確認する
* 仮想マシンを立ち上げる
  ```
  $ vagrant up
  ```
  またはapサーバのみ立ち上げるならば以下
  ```
  $ vagrant up ap
  ```
* sshで仮想マシンにログインする
  ```
  $ vagrant ssh ap
  ```

### 1.wgetコマンドのインストール
* wgetのインストールをする
  ```
  $ sudo yum install -y wget
  ```
  デフォルトではwgetコマンドがインストールされていない。PostgreSQLのRPMパッケージをダウンロードするためにwgetコマンドをインストールする。

### 2.PostgreSQLのインストール
* PostgreSQL10のRPMパッケージをダウンロードする
  ```
  $ cd /tmp
  $ wget https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm
  ```
  バージョンを指定してyumでインストールするためrpmパッケージを一度取得している。
* ダウンロードしたRPMパッケージのインポート
  ```
  $ sudo yum localinstall -y pgdg-centos10-10-2.noarch.rpm
  ```
* PostgreSQLのインストールを行う
  ```
  $ sudo yum install -y postgresql10
  $ sudo yum install -y postgresql10-libs
  $ sudo yum install -y postgresql10-server
  ```
  対象は本体、ライブラリ、サーバ。

### 3.PostgreSQLの初期化
* setup用のスクリプトを実行し初期化を行う。
  ```
  $ cd /usr/pgsql-10/bin
  $ sudo ./postgresql-10-setup initdb
  ```

### 4.PostgreSQLへの認証方式変更
* ローカル（コマンド）からのアクセスに対し認証を行わないように変更する。
  ```
  $ sudo vi /var/lib/pgsql/10/data/pg_hba.conf
  ```
  ファイル下部を以下のように変更する。（この資料上スペースの数が短縮されて表示されているかもしれない。）
  - 変更前: `local   all             all                                     peer`
  - 変更後: `local   all             all                                     trust`
* ローカルからのIPv4によるアクセスに対しパスワード認証をするように変更する。これはtomcatからの接続を行うための設定。
  ```
  $ sudo vi /var/lib/pgsql/10/data/pg_hba.conf
  ```
  ファイル下部を以下のように変更する。（この資料上スペースの数が短縮されて表示されているかもしれない。）
  - 変更前: `host    all             all             127.0.0.1/32            ident`
  - 変更後: `host    all             all             127.0.0.1/32            password`


### 5.サービス自動起動設定を行う
* 自動起動が有効か確認する
  ```
  $ systemctl is-enabled postgresql-10
  ```
* 自動起動を有効にする
  ```
  $ sudo systemctl enable postgresql-10
  ```
* 有効になっているか確認する
  ```
  $ systemctl is-enabled postgresql-10
  ```

### 6.サービスの起動（再起動）をする。
* サービスの起動状態を確認する
  ```
  $ systemctl is-active postgresql-10
  ```
* サービスの起動（再起動）をする
  ```
  $ sudo systemctl start postgresql-10
  ```
  再起動の場合は
  ```
  sudo systemctl restart postgresql-10
  ```
* サービスの起動状態を確認する
  ```
  $ systemctl is-active postgresql-10
  ```

### 7.databaseの作成
* アプリで使用するデータベースを作成する
  ```
  $ psql -U postgres
  ```
  上記のコマンドを実行すると対話モードに切り替わる。（以降psqlコマンドの対話モードは `#` から記述して表すとする。）
  ```
  # CREATE DATABASE example TEMPLATE template0 ENCODING UTF8;
  ```
* データベースが作成されたこと確認する。
  ```
  # \l
  ```
* 対話モードを終了する（以降は本手順を記載しない）
  ```
  # \q
  ```

### 8.ユーザの作成
* 作成したデータベースを対象に接続する。
  ```
  $ psql -U postgres example
  ```
* アプリ用のユーザを作成する。
  ```
  # CREATE USER nablarch PASSWORD 'password123';
  ```
* ユーザが作成されていることを確認する。
  ```
  # \du
  ```
* (ユーザのパスワードが正しく設定されていることを確認する)
  ```
  $ psql -U nablarch example -h 127.0.0.1
  ```
  パスワードが求められるので設定したパスワードを入力し対話モードに切り替われば正常に完了している。

  パスワードを求められないなどがあれば「4.PostgreSQLへの認証方式変更」の内容を間違えていたり読み込みされていない可能性が考えられる。

### 9.スキーマの作成とpublicスキーマの削除
* アプリ用のスキーマを作成する。オーナーはアプリ用のユーザ。
  ```
  $ psql -U postgres example
  # CREATE SCHEMA nablarch AUTHORIZATION nablarch;
  ```
  スキーマはユーザと別名を設定できるしその場合上記のコマンドを短縮できるが、以下の理由により上記のようなコマンドにしている。
  - ユーザと同名のものをデフォルトスキーマとするDB製品がある
  - コマンドは短縮系よりも一度短縮していない状態のものを使いたかった
* スキーマの確認をする。
  ```
  # \dn
  ```
* publicスキーマの削除。
  ```
  # DROP SCHEMA public;
  ```
  データベース作成時にあらかじめ用意される全体向けのスキーマだが、以下の理由により削除。
  - 使用しない
  - publicスキーマがあるとカレントスキーマの変更をしないとならない。
  - tomcatからアプリユーザを使用した時にユーザにsearchパスの設定をしなければならない。
* スキーマの確認をする。
  ```
  # \dn
  ```

### 10.DDLの実行
* DDLを実行しアプリに必要なテーブル・データを用意する。
  ```
  $ psql -f /vagrant/ap/resources/create.sql -U postgres -d example
  ```
  このコマンドの結果内容確認は次の手順で行う。ここではエラーがなければ良い。

### 11.ユーザへテーブルの権限を付与する
* アプリユーザへアプリ用スキーマにある全テーブルの権限（参照、登録、更新、削除）を付与する
  ```
  $ psql -U postgres example
  # GRANT SELECT,INSERT,UPDATE,DELETE ON ALL TABLES IN SCHEMA nablarch TO nablarch;
  ```
* 権限の確認をするためアプリユーザで接続し直す。
  ```
  # \connect - nablarch
  ```
* それぞれのテーブルに対する権限があることを表示する
  ```
  # \z
  ```
* 参照だけしてみる
  ```
  # SELECT * FROM CODE_NAME;
  ```
