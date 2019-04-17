# ここは何をする場所か

Jenkinsを好きに触って好きに捨てれるようにした場所

単純にJenkinsの操作を学ぶため作りました。

`vagrant destroy` するとプラグインとか設定とか全部消えちゃうので悲しい気持ちになる。

# 始める

## VMの立ち上げ

* `vagrant up` を実行。（jenkinsのインストールとか全部やってくれる）
* 標準出力の最後の方に出力される以下の部分を控えること

|何に使うか|出力されるタスク|使う文字列の例|備考|
|-|-|-|-|
|jenkinsのアンロックパスワード|Print init password Jenkins|`"passwd.stdout": "xxxxxxxxxxx"` のxxxxxxxxxxxとなっている部分|jenkinsのアンロック後は出力されなくなる|
|jenkinsで使うJDKの登録|Print java command path|`"javapath.stdout": "/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.xxx.xxxxxxxxxx.x86_64/jre/bin/java"` パス部分で `/jre/bin/java` の前の部分まで||
|jenkinsで使うMavenの登録|Print mvn home directory|`"mvnpath.stdout": "Maven home: /usr/share/maven"` パス部分|これは `mvn -v` で出力される結果から取得しているため他のものと形式が異なる|

## Jenkinsの立ち上げ
* `http://localhost:38080/` にアクセス
* アンロックパスワードを入力する
* プロキシを利用する場合パスワード入力後に入力できる
* `Install suggest plugin` を選ぶといい感じにインストールされる（プロキシ環境は知らない）
* ユーザは適当に作る

## 初期設定
* 作ったユーザでログイン
* Jenkinsの管理 -> Global Tool Configuration からJDKとMavenのhomeを入力する


# 参考にしたところ

* https://weblabo.oscasierra.net/jenkins-install-centos7/
* https://qiita.com/kota344/items/304979feaf965afffb1a
* https://awsbloglink.wordpress.com/2018/10/08/ansible%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%A6jenkins%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95-2/