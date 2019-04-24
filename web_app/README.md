# これは何か

Webサーバ - APサーバ - DBサーバを学ぶ場所

よくあるwebアプリケーションの3層構造をなんとなく実現できるようになることを目標にする。

# 環境構成

サーバはWebサーバ(apache)とAPサーバ(tomcat + PostgresSQL)の２台

想定としてWebサーバはDMZに置かれるがAPサーバはLAN(internal)に置かれるだろうという想定のため今回はAPサーバにDBを同居させた

|役割|host名|IPアドレス|確認用ポートフォワーディングの設定|
|-|-|-|-|
|Webサーバ|web|192.168.33.10|ローカルマシン:30080 -> VM:80 (http)|
|APサーバ|ap|192.168.33.11|ローカルマシン:38080 -> VM:8080 (http)|

## Webサーバ

* プライベートIPアドレス: `192.168.33.10`
* 解放するポートは22(ssh)と80(http)のみ（ファイアウォールで許可するもの）
* `/example/action/*` のリクエストをリバースプロキシでAPサーバへ送る(apacheにリバースプロキシの設定を入れる)
* `/example/css/*` と `/example/js/*` のリクエストはWebサーバのリソースを返却する

## APサーバ

* プライベートIPアドレス: `192.168.33.11`
* 解放するポートは22(ssh)と8080(http)のみ（ファイアウォールで許可するがプライベートネットワークからの接続のみ受け付ける）
* tomcatからDBへの接続はtomcatに設定のjndiで

apache + tomcatの構成ならHTTPでなくAJPで通信した方が良いかと思ったが接続確認をwebサーバを介さずに行いたかった。


# 参考
## Apacheの参考
### プロキシ（リバースプロキシ）関連
* http://webos-goodies.jp/archives/51261261.html
* https://httpd.apache.org/docs/2.4/ja/mod/mod_proxy.html
### セクション関連
* https://httpd.apache.org/docs/2.4/ja/sections.html

## Tomcatの参考
* https://qiita.com/msakamoto_sf/items/ef34922548275dd31014

## CentOS7のサービスについて
* https://www.server-memo.net/centos-settings/centos7/service-start.html

## SELinuxについて
* https://blog.fenrir-inc.com/jp/2016/09/selinux.html
* https://docs.ansible.com/ansible/latest/modules/seboolean_module.html

## PosgreSQLの参考
### 初期構築
* http://serverarekore.blogspot.com/2018/09/ansibleapache-nifipostgresql10centos75.html
* https://qiita.com/LowSE01/items/84af05449f96dedd0edc
### 認証設定
* https://www.postgresql.jp/document/10/html/auth-pg-hba-conf.html
