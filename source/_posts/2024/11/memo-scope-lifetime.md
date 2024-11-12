---
title: ipコマンドの出力結果（scope、lifetime）
date: 2024-11-12 19:25:48
tags: TCP/IP
category: network
---

[前回の投稿](https://tkaizawa.github.io/myblog/memo-localhost-ip-and-loopback/)で`ip a`コマンドを見た際、ループバックアドレス以外のアドレスの内容も気になり調べた結果をメモしておく。

例えば、以下のような出力結果について見ていく。

~~~
inet 133.xxx.xxx.xxx/24 brd 133.xxx.xxx.255 scope global noprefixroute enp1s0
   valid_lft forever preferred_lft forever
~~~

<br>

---
# scopeについて
---

[前回の投稿](https://tkaizawa.github.io/myblog/memo-localhost-ip-and-loopback/)では`scope local`がローカルホスト内でのみ有効で、外部アクセスはできない設定だと学んだ。

そもそもこの**scope**はIPネットワークの範囲（IPアドレスが有効な範囲、アクセス可能範囲）を表現した概念。

scopeにはlocalのほかにglobal、linkなどの分類がある（IPv4、IPv6によって定義が異なる）。

`scope global`：
- グローバルにアクセス可能なアドレス。
- 例えば、インターネットにアクセスするための`133.xxx.xxx.xxx`などのIPv4アドレスなど。

`scope link`：
- 同一リンク（ネットワークインターフェース）上でのみ有効なアドレスで、ルータ等のL3中継装置で区切られたセグメント単位を越えることはできない。
- 主にリンクローカルアドレス（IPv6の`fe80::/64`範囲など）で使用され、隣接デバイスとの直接通信に使われる。

【参考】[スコープという考え方](https://www.vwnet.jp/IPv6ImplementationGuide/01/01-11.htm)

<br>

---
# valid_lftについて
---

`valid_lft`や`preferred_lft`などはIPアドレスの有効期間（Lifetime）を示している。

`valid_lft`：
- Valid_Lifetimeの略で、アドレスが有効である期間を秒数で表示する。
- `forever`と表示される場合、アドレスの有効期限が無期限であることを表す。
- この時間が切れるとそのアドレスは無効になり、システムから削除されることがある。

`preferred_lft`：
- Preferred_Lifetimeの略で、アドレスが優先的に使用される期間を示している。
- この期間内は、システムが他のアドレスよりもこのアドレスを優先して使う。

<br>

---
# 出力結果の再見
---

再度出力結果を見てみる。

~~~
inet 133.xxx.xxx.xxx/24 brd 133.xxx.xxx.255 scope global noprefixroute enp1s0
   valid_lft forever preferred_lft forever
~~~
<br>

この例の意味は以下のようになる。

`enp1s0`：インターフェース名（接続名）
`scope global`：グローバルにアクセス可能である
`valid_lft forever preferred_lft forever`：このアドレスは無期限に有効で、優先的に使用される

なお、`ip`コマンドの内容の見方（`ifconfig`を使う人もいるが[非推奨らしい](https://tech.mktime.com/entry/211)）については、以下の方が素晴らしくまとめて下さっている。
[ifconfigの出力結果に書いてあること](https://qiita.com/pe-ta/items/aff8db72530c6baa11b2)


