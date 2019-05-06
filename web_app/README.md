# これは何か

Webサーバ - APサーバ - DBサーバを学ぶ場所

よくあるwebアプリケーションの3層構造をなんとなく実現できるようになることを目標にする。

`vagrant up` したら `http://localhost:30080/exam-app` で接続確認する。

# 環境構成

サーバはWebサーバ(apache)とAPサーバ(tomcat + PostgresSQL)の２台

想定としてWebサーバはDMZに置かれるがAPサーバはLAN(internal)に置かれるだろうという想定のため今回はAPサーバにDBを同居させた

APサーバのポートフォワーディングの設定はtomcatの動作確認用。

|役割|host名|IPアドレス|確認用ポートフォワーディングの設定|
|-|-|-|-|
|Webサーバ|web|192.168.33.10|ローカルマシン:30080 -> VM:80 (http)|
|APサーバ|ap|192.168.33.11|ローカルマシン:38080 -> VM:8080 (http)|

## Webサーバ

* プライベートIPアドレス: `192.168.33.10`
* 解放するポートは22(ssh)と80(http)のみ（ファイアウォールで許可するもの）
* `/example/action/*` のリクエストをリバースプロキシでAPサーバへ送る(apacheにリバースプロキシの設定を入れる)
* `/example/css/*` と `/example/js/*` のリクエストはWebサーバのリソースを返却する(予定)
* SELinuxは無効にするのではなくのapacheからネットワークへの接続のみ許可

## APサーバ

* プライベートIPアドレス: `192.168.33.11`
* 解放するポートは22(ssh)と8080(http)のみ（ファイアウォールで許可するがプライベートネットワークからの接続のみ受け付ける）
* tomcatからDBへの接続はtomcatに設定のjndiで
* tomcatの起動パラメータで言語を日本語に設定
* DBサーバはPostgreSQL
* アプリ用のユーザ、同名のスキーマを作成し、テーブルへの閲覧・登録・更新・削除権限を与えている
* SELinuxは無効にするのではなくのtomcatからDBへの接続のみ許可

apache + tomcatの構成ならHTTPでなくAJPで通信した方が良いかと思ったが接続確認をwebサーバを介さずに行いたかった。

# 練習してみる

1. feature-trainingブランチをチェックアウトする
    - `git checkout -b feature-training origin/feature-traing`
2. `vagrant up` する
3. ポートフォワーディングとプロキシの設定のみされるので以下の資料を元に練習する。
    - [webサーバ(apache)](./docs/apache_handson.md)
    - [APサーバ(tomcat)](./docs/tomcat_handson.md)
    - [DBサーバ(PostgreSQL)](./docs/postgresql_handson.md)

# 参考
## Apacheの参考
### プロキシ（リバースプロキシ）関連
* http://webos-goodies.jp/archives/51261261.html
* https://httpd.apache.org/docs/2.4/ja/mod/mod_proxy.html
### セクション関連
* https://httpd.apache.org/docs/2.4/ja/sections.html

## Tomcatの参考
### yumのパッケージ関連
* https://qiita.com/msakamoto_sf/items/ef34922548275dd31014
### JNDI関連
* https://tomcat.apache.org/tomcat-8.0-doc/jndi-datasource-examples-howto.html
* https://qiita.com/nekijak/items/f00c615f0e84bc647c02
* https://www.infoscoop.org/blogjp/2014/12/04/no-basicdatasourcefactory/

## CentOS7参考
### サービスについて
* https://www.server-memo.net/centos-settings/centos7/service-start.html
* https://qiita.com/ao_log/items/78e6f620ec07cbac47a1

### SELinuxについて
* https://blog.fenrir-inc.com/jp/2016/09/selinux.html
* https://docs.ansible.com/ansible/latest/modules/seboolean_module.html
* https://qiita.com/yunano/items/857ab36faa0d695573dd

## PosgreSQLの参考
### 初期構築
* http://serverarekore.blogspot.com/2018/09/ansibleapache-nifipostgresql10centos75.html
* https://qiita.com/LowSE01/items/84af05449f96dedd0edc
* https://qiita.com/ponsuke0531/items/12bf73640ee54c238e7e
### 認証設定
* https://www.postgresql.jp/document/10/html/auth-pg-hba-conf.html
* https://qiita.com/k1LoW/items/7d3ed522286fba421de3
### コマンド
* https://dev.classmethod.jp/server-side/db/postgresql-organize-command/
* https://qiita.com/mm36/items/1801573a478cb2865242
