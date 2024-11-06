---
title: wiregurad設定失敗
date: 2024-11-06 10:56:14
tags: 
 - ssh
 - network
 - wireguard
category: tech
---

[某テストベッド](https://it.impress.co.jp/category/c320097)に参加するためのネットワーク設定が上手くいかない。

色々試した記録を残す。

<br>

## <u>構成</u>

wireguardを使って主催者側の閉域網（10.250.*.*）に入る。

参加者はubuntuを2台用意し、1台にwireguardクライアントを入れてルータとして機能させ、もう一台はルータ端末にぶら下がる形でネットワークに接続できるよう設定する。

２パターンでの接続を試している。

1. 職場のプライベートクラウドからの接続
2. パブリッククラウド（OCI）からの接続

<br>

## <u>1. 職場のプライベートクラウドからの接続</u>

### <u>1-1. ルータ端末の設定</u>

主催者側のドキュメントに従い、wireguardの設定を進める。
事前に受け取っていた.confファイルの中身を一部編集し、wireguardクライアントを起動。

~~~
$ sudo apt update
$ sudo apt install wireguard

$ sudo vi /etc/wireguard/wg0.conf #ファイルアクセス権限がない場合はchmod

$sudo wg-quick up wg0
$sudo systemctl enable wg-quick@wg0
~~~
<br>

接続確認のためnslookupとcurlを実行したところ、こちらは無事に疎通できた。

~~~
> nslookup xxxxxx
Server: 123.456.789.0
Address: 123.456.789.0#0

Non-authoritative answer:
Name: xxxxxx
Address: 123.456.789.0

> curl xxxxxx
~~~
<br>

### <u>1-2. ぶらさがる端末の設定</u>

問題はこちら。

`/etc/netplan/99-xxxx-xxxx.yaml`を作成し、指定の内容で静的ルーティングの設定を行う。

~~~
$ sudo vi /etc/netplan/99-xxxx-xxxx.yaml

$ sudo netplan apply
~~~
<br>

ルータ端末と同じようにnslookupを行うも、疎通できず。

~~~
> nslookup xxxxxx
Server: 123.456.789.0
Address: 123.456.789.0#0

** server can't find xxxxxx: NXDOMAIN
~~~
<br>

pingでどこが通っているか試してみると、ルータ端末のぶら下がり端末の間は疎通できているらしい。

ということは、ルータ端末のルーティング設定が上手くいっていない？

<br>

### <u>試行１：ブリッジ用のNICを追加</u>

ブリッジインターフェースと固定IPアドレスを設定して、両端末にインターフェイスを追加してみる。
そのために、ルータ側とぶら下がり側の両方に、ブリッジ用の設定ファイル（60-add.yaml）を設置してnetplan applyを実行。

結果は、、、、インスタンス自体に接続できなくなってしまったヽ(´Д｀;≡;´Д｀)ﾉ

クラウド管理の御仁に詫びを入れ、コンソールから設定ファイルを削除してもらい、復旧。

<br>

### <u>試行2：1つのNICに複数の固定IPアドレスを設定してみる</u>

参考にさせていただいたのは[こちら](https://qiita.com/rat-engineer755/items/b04e128ee1d2cb4437a5)

まず、cloud-initを無効化のためのファイル作成。

~~~
$ sudo vi /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

# sudo touch /etc/cloud/cloud-init.disabledでも良い？
~~~
<br>

現在の設定のバックアップ。

~~~
$ cp  /etc/netplan/50-cloud-init.yaml ~/50-cloud-init.yaml.bak
~~~
<br>

NIC設定の編集。

~~~
$ vi /etc/netplan/50-cloud-init.yaml
~~~
<br>

で、ルータ端末とぶら下がり端末の両方に10.250.x.y.（主催者側から割り当てられたアドレスに対応するもの）を`address`に追記。

さらに、外部からのアクセス確保のため、`routes:`に以下を追記。

~~~
routes: 
              - to:  defalut
                via: aaa.aaa.aaa.aaa 
~~~
<br>

で、設定を適用。

~~~
$ sudo netplan apply
~~~
<br>
これで再びnslookupを試してみるが、、、、

~~~
** server can't find xxxxxx: NXDOMAIN
~~~
<br>

やっぱりダメ。

<br>

### <u>試行3：`dhcp4-overrides`オプションを使用</u>

主催者側に相談してみたところ、以下を/etc/netplan/99-xxxx-xxxx.yamlに追記してみてほしいとのこと。

~~~
dhcp4-overrides:
    use-dns: false
    use-domains: false
~~~
<br>

これを追記して、netplan applyからのnslookup。結果は、、、

~~~
** server can't find xxxxxx: NXDOMAIN
~~~
<br>

これもダメ。

<br>

## <u>1. 職場のプライベートクラウドからの接続</u>

クラウド管理の御仁に相談したところ、OCIで10.250のプライベートネットワークを組んでubuntuを3台（踏み台＋実機）立ててくださった。大感謝！！

これでできるのではと、意気揚々とドキュメント通りに作業を進めたところ、、、、

wireguardのクライアント起動時のエラー（`/usr/bin/wg-quick: line 32: resolvconf: command not found`）に対処するシンボリックリンク作成コマンドを実行したら、なんとインスタンスが固まってしまい、またsshできなくなる。。。

~~~
$ sudo ln -s /usr/bin/resolvectl /usr/local/bin/resolvconf
~~~
<br>

またまたコンソールからリンクを削除してもらう羽目に。すみません。

なかなか環境構築が進まない。。。

<br>