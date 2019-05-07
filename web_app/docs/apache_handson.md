# Apache練習

## ここでやること
* CentOS7にApacheをインストールする
* SELinuxの許可設定を行う
* リバースプロキシの設定を行う
* サービス自動起動設定を行う
* サービスの起動(再起動)をする。
* 諸々の確認作業

## やってみる

### 0.環境構築〜ログインする
* 仮想マシンを立ち上げるためカレントディレクトリが `web_app` であることを確認する
* 仮想マシンを立ち上げる
  ```
  $ vagrant up
  ```
  またはwebサーバのみ立ち上げるならば以下
  ```
  $ vagrant up web
  ```
* sshで仮想マシンにログインする
  ```
  $ vagrant ssh web
  ```

### 1.CentOS7にApacheをインストールする

初期状態ではApacheがインストールされていないためインストールする。

* yumでapacheをインストールする
  ```
  $ sudo yum install -y httpd
  ```
* サービスに登録されていることを確認する
  ```
  $ systemctl list-unit-files --type service | grep httpd
  ```

### 2.SELinuxの許可設定を行う

これが許可されていないとリバースプロキシの設定をしてもapacheからtomcatなどへ接続できない。

* apacheの許可状態を調べる
  ```
  $ getsebool -a | grep httpd_can
  ```
  今回の対象は `httpd_can_network_connect` のみ
* 許可設定をする
  ```
  $ sudo setsebool httpd_can_network_connect on
  ```
* 再度確認する
  ```
  $ getsebool -a | grep httpd_can
  ```

### 3.リバースプロキシの設定を行う
* Apacheのプロキシモジュールが読み込まれていることを確認する。
  ```
  $ cd /etc/httpd/
  $ grep mod_proxy *
  ```
  今回は `mod_proxy.so` と `mod_proxy_http.so` がコメントアウトされていないことを確認できれば良い
* リバースプロキシ用の設定ファイルを作成する
  ```
  $ cd /etc/httpd/conf.d/
  $ sudo vi ap_proxy.conf
  ```
* 以下の内容を設定し保存する
  ```
  ProxyRequests Off
  <Location "/exam-app">
    Order deny,allow
    Allow from all
  </Location>

  ProxyPass /exam-app http://192.168.33.11:8080/exam-app
  ProxyPassReverse /exam-app  http://192.168.33.11:8080/exam-app
  ```

### 4.サービス自動起動設定を行う
* 自動起動が有効か確認する
  ```
  $ systemctl is-enabled httpd
  ```
* 自動起動を有効にする
  ```
  $ sudo systemctl enable httpd
  ```
* 有効になっているか確認する
  ```
  $ systemctl is-enabled httpd
  ```

### 5.サービスの起動（再起動）をする。
* サービスの起動状態を確認する
  ```
  $ systemctl is-active httpd
  ```
* サービスの起動（再起動）をする
  ```
  $ sudo systemctl start httpd
  ```
  再起動の場合は
  ```
  sudo systemctl restart httpd
  ```
* サービスの起動状態を確認する
  ```
  $ systemctl is-active httpd
  ```

### 6.諸々の確認作業
* ローカルマシンのブラウザから `http://localhost:30080/` と `http://localhost:30080/exam-app/` にアクセスしてみる
* apacheのアクセスログを確認
  ```
  $ sudo view /var/log/httpd/access_log
  ```
  ログの読み方は [公式](https://httpd.apache.org/docs/2.4/ja/logs.html) を参照
* サービス起動時のログ確認
  ```
  $ sudo journalctl -u httpd
  ```