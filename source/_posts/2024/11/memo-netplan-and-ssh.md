---
title: netplanとsshのメモ
date: 2024-11-05 12:51:50
tags: 
 - ssh
category: network
---

職場のクラウド環境から外部のプライベート環境に行くための設定でミスったので、備忘のためのメモ。

### <u>やってしまったこと</u>
ubuntuのnetplanにネットワーク設定を追加するyamlファイル（60-xxx.yaml）を置いたところ、sshができなくなってしまった。

クラウドを管理してくださってる方にお願いして、yamlファイルを削除してもらったところ、無事復旧。

<br>

###  <u>sshの超初歩ミス</u>
その後、別のパブリッククラウドもubuntu2台を構築してもらっているが、アドレスを指定してもアクセスできない。

と騒いでいたら、ユーザ名を入れていないだけだと分かり赤面。

~~~
$ ssh aizawa@123.456.789.0
~~~
<br>

踏み台サーバから２台に入るための設定は/etc/hostsに追記してくださっているので、サーバ名だけ入れれば2台とも入れる。

~~~
$ cat /etc/hosts

$ ssh cadde-xxx #踏み台サーバに入っている状態で 
~~~
<br>

.ssh/configを書けば一発で入れるとのこと。たぶんこういうのだと思うので、後で試してみる。
[SSH接続先の情報をssh configに記載する
](https://qiita.com/miriwo/items/976d21e9a42fd3b01d08)

<br>

