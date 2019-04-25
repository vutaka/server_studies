# これは何

なんとなくサーバとかの勉強をする場所

# 環境構築

* VirtualBox 6.0
* Vagrant 2.2.4(1.9以降なら多分動く)

windowsでdockerを動かすのがめんどくさかったので。。。

基本的にCentOS7を動かします。

## プロキシを使用する場合

プロキシを使用する場合は必ずプラグインをインストールすること

`vagrant plugin install vagrant-proxyconf`

ここにあるVagrantfileは `http_proxy` という環境変数からプロキシを設定するようにしているため、 プロキシを利用する場合必ず `http_proxy` を設定すること

## windowsで動かすときの注意

### `Encoding::InvalidByteSequenceError` とかエラーが出る。

コマンドプロンプトかPSの文字コードをUTF-8変更した上で実行する。

* `chcp 65001`

### `Encoding::CompatibilityError` とかエラーが出る。

Vagrantfileを置いているパスに日本語が含まれているはずなので全て英語にする。


# 基本的にやってること

* 使っているOSはCentOS7
* プロビジョナはシェルを書くのが辛いのでansible_local

# 参考
## ansible
* https://docs.ansible.com/ansible/latest/modules/modules_by_category.html