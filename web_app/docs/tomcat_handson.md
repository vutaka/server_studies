# Tomcat構築練習
## 前提

* DBの構築が完了していること。

## アプリ情報

tomcatの構築を練習する場所

ここで使用するアプリケーションは以下のもののwarをリネームしたもの。(コンテキストルートがwarの名称になるためリネームしている。)

https://github.com/vutaka/example_project

war作成コマンド：`mvn -P gsp,prod package`

## tomcatの構成

tomcatはCATALINA_BASEを分けることでサーバで複数プロセス立ち上げることができる。（別のwebアプリを複数立ち上げれる。）

ただし本サーバでは行なっていない。

最低限の文言の説明だけ行う
* CATALINA_HOME: tomcatの本体をインストールした場所
* CATALINA_BASE: webアプリを動かす場所。最低限のファイルのみで動かせる。

## ここでやること
* サーバの言語・タイムゾーンの設定
* 必要なパッケージのインストール
* tomcatサンプルアプリの削除
* アプリ用のディレクトリ作成
* JDBCドライバダウンロード（PostgreSQL用）
* JNDIの設定
* tomcatの言語設定
* SELinuxの設定（tomcatからDB）
* アプリのデプロイ（warの配置）
* tomcatの自動起動の設定
* tomcatのサービス開始
* 起動確認
* firewalldの設定ファイル作成(webサーバの構築が終わった後に行うこと)
* firewalldの設定有効化

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

### 1.サーバの言語・タイムゾーンの設定
* サーバの言語を日本語（UTF-8）にする。
  ```
  $ sudo localectl set-locale LANG=ja_JP.UTF-8
  ```
  デフォルトでは英語設定などになっておりアプリログが文字化けなどを起こす。
  ここで設定した言語が必ずしもtomcatのデフォルト言語にはならないためtomcatには別途設定する。
* 言語の確認をする
  ```
  $ localectl
  ```
* タイムゾーンの設定
  ```
  $ sudo timedatectl set-timezone Asia/Tokyo
  ```
  デフォルトではUTC。ログの出力時間がずれるため設定する。
* タイムゾーンの確認をする。
  ```
  $ timedatectl
  ```

### 2.必要なパッケージのインストール
* OpenJDK8のインストールをする
  ```
  $ sudo yum install -y java-1.8.0-openjdk
  ```
* tomcat、tomcatのwebapp用パッケージ、tomcatのweapp管理用パッケージのインストール
  ```
  $ sudo yum install -y tomcat
  $ sudo yum install -y tomcat-webapps
  $ sudo yum install -y tomcat-admin-webapps
  ```
* wgetのインストール
  ```
  $ sudo yum install -y wget
  ```

### 3.tomcatサンプルアプリの削除
* tomcatのwebapp用パッケージに入っているサンプルアプリを削除する。
  ```
  $ sudo rm -rd /usr/share/tomcat/webapps/*
  ```

### 4.アプリ用のディレクトリ作成
* tomcatのサービス情報確認
  tomcatの起動ユーザの確認をする。`User=tomcat`となっていること。
  ```
  $ cat /usr/lib/systemd/system/tomcat.service
  ```
* tomcatの所属グループを確認する
  所属グループもtomcatになっていること。
  ```
  $ id tomcat
  ```
* アプリが使用するディレクトリを作成する。
  ```
  $ sudo mkdir -p /var/nablarch/output
  $ sudo mkdir -p /var/nablarch/format
  ```
* ディレクトリの所有者・所有グループを変更する
  ```
  $ sudo chown -R tomcat:tomcat /var/nablarch
  ```
* ディレクトリの権限を変更する
  ```
  $ sudo chmod -R 774 /var/nablarch/
  ```

### 5.JDBCドライバダウンロード（PostgreSQL用）
* tomcatのライブラリ置き場に移動する
  ```
  $ cd /usr/share/tomcat/lib/
  ```
* wgetでJDBCドライバダウンロード
  ```
  $ sudo wget https://jdbc.postgresql.org/download/postgresql-42.2.5.jar
  ```

### 6.JNDIの設定
* 設定ファイルのあるディレクトリに移動する。
  ```
  $ cd /usr/share/tomcat/conf/
  ```
* 設定ファイルを編集する
  ```
  $ sudo vi context.xml
  ```
* `</Context>` 前に以下の行を追加する
  ```
  <Resource
    name="datasource"
    auth="Container"
    type="javax.sql.DataSource"
    driverClassName="org.postgresql.Driver"
    url="jdbc:postgresql://127.0.0.1:5432/example"
    factory="org.apache.commons.dbcp.BasicDataSourceFactory"
    username="nablarch"
    password="password123"
    maxTotal="20"
    maxIdle="10"
    maxWaitMillis="-1"/>
  ```

### 7.tomcatの言語設定
* 設定ファイルを配置するディレクトリに移動する。
  ```
  $ cd /usr/share/tomcat/conf/conf.d
  ```
* 設定ファイルを作成する。
  ```
  $ sudo vi lang.conf
  ```
* 以下の内容を記載する
  ```
  CATALINA_OPTS="-Duser.language=ja -Duser.country=JP"
  ```
* 所有者、権限などを変更する
  ```
  $ sudo chown tomcat:tomcat lang.conf
  $ sudo chmod 774 lang.conf
  ```

### 8.SELinuxの設定（tomcatからDB）

これが許可されていないとtomcatからdbへ接続できない。

* tomcatの許可状態を調べる
  ```
  $ getsebool -a | grep tomcat
  ```
  今回の対象は `tomcat_can_network_connect_db` のみ
* 許可設定をする
  ```
  $ sudo setsebool -P tomcat_can_network_connect_db on
  ```
* 再度確認する
  ```
  $ getsebool -a | grep tomcat
  ```

### 9.アプリのデプロイ（warの配置）

基本的にtomcatはwarファイルを所定の場所に移動することで自動的に展開される（ `/usr/share/tomcat/conf/server.xml` にそういう設定がある。）

* warファイルを移動する。
  ```
  $ sudo cp /vagrant/ap/resources/exam-app.war /usr/share/tomcat/webapps/
  ```
* 権限を変更する
  ```
  $ sudo chmod 644 /usr/share/tomcat/webapps/exam-app.war
  ```

### 10.tomcatの自動起動の設定
* 自動起動が有効か確認する
  ```
  $ systemctl is-enabled tomcat
  ```
* 自動起動を有効にする
  ```
  $ sudo systemctl enable tomcat
  ```
* 有効になっているか確認する
  ```
  $ systemctl is-enabled tomcat
  ```

### 11.tomcatのサービス開始
* サービスの起動状態を確認する
  ```
  $ systemctl is-active tomcat
  ```
* サービスの起動（再起動）をする
  ```
  $ sudo systemctl start tomcat
  ```
  再起動の場合は
  ```
  $ sudo systemctl restart tomcat
  ```
* サービスの起動状態を確認する
  ```
  $ systemctl is-active tomcat
  ```

### 12.起動確認
* 以下のファイルを確認し正常に起動していることを確認する
  ```
  $ view /usr/share/tomcat/app.log
  ```
  起動に失敗している場合ログから原因を解消すること
* ここでもし対象のファイルがなかった場合はtomcat自体のログを確認する必要がある。以下の場所にあるファイルの日時が最新のものを確認すること
  ```
  $ sudo ls -tl /var/log/tomcat
  ```

### 13.firewalldの設定ファイル作成(webサーバの構築が終わった後に行うこと)
* 以下のコマンドをapサーバで発行し正常にレスポンスが帰ってくることを確かめる。
  ```
  $ curl http://localhost:8080/exam-app/
  ```
* ローカルマシンのブラウザからwebサーバ経由でアクセスし、アクセスできないことを確認する。
  次のURLにアクセス `http://localhost:30080/exam-app/`

  ファイアウォールにより拒否されているため外部からのアクセスができないことがわかる。
* ファイアウォールの設定ファイルを配置する場所に移動する
  ```
  $ cd /usr/lib/firewalld/services/
  ```
* tomcat用の設定ファイルを作成する。
  ```
  $ sudo vi tomcat.xml
  ```
* 以下の内容を記載する
  ```
  <?xml version="1.0" encoding="utf-8"?>
  <service>
    <short>Tomcat</short>
    <description>Open Tomcat port</description>
    <port protocol="tcp" port="8080"/>
  </service>
  ```

### 14.firewalldの設定有効化
* firewalldの起動
  ```
  $ sudo systemctl start firewalld
  ```
* firewalldの登録ずみサービスの確認
  ```
  $ sudo firewall-cmd --list-services
  ```
  tomcatが登録されていないことを確認する
* tomcatを永続化で登録し読み込み直す。
  ```
  $ sudo firewall-cmd --add-service=tomcat --permanent
  $ sudo firewall-cmd --reload
  ```
* ローカルマシンのブラウザからwebサーバ経由でアクセスし、アクセスできることを確認する。
  次のURLにアクセス `http://localhost:30080/exam-app/`
