# ここは何をする場所か

Jenkinsを好きに触って好きに捨てれるようにした場所

単純にJenkinsの操作を学ぶため作りました。

`vagrant destroy` するとプラグインとか設定とか全部消えちゃうので悲しい気持ちになる。

# 始める

* `vagrant up` を実行。（jenkinsのインストールとか全部やってくれる）
* コマンド完了後 `http://localhost:38080/` にアクセス
* アンロックパスワードはコマンド実行時に表示される `result.stdout` の後の文字列
* `Install suggest plugin` を選ぶといい感じにインストールされる（プロキシ環境は知らない）

本当にdockerにしておけばよかった。

# 参考にしたところ

* https://weblabo.oscasierra.net/jenkins-install-centos7/
* https://qiita.com/kota344/items/304979feaf965afffb1a
* https://awsbloglink.wordpress.com/2018/10/08/ansible%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%A6jenkins%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95-2/