---
title: wiregurad設定成功メモ
date: 2024-11-07 10:56:14
tags: 
- network
- wireguard
category: tech
---

ようやくwireguardの設定がうまくいったので、メモに残す。
<br>

---
# 手順
---

<!-- toc -->

## <u>1. NICに固定IPアドレスを追加</u>

[失敗メモ]（https://tkaizawa.github.io/myblog/2024/11/wireguard-setting-failure/）で行っていた、固定IPをNICに設定。

<br>

## <u>2. DNSを指定して名前解決を実行</u>

[失敗メモ]（https://tkaizawa.github.io/myblog/2024/11/wireguard-setting-failure/）ではこの時点でnslookupコマンドが通らなかったが、明示的にDNSを指定して試してみたところ、無事名前解決ができた。

~~~
$ nslookup xxxxxx 10.250.aaa.aaa

Non-authoritative answer:
Name:	xxxxxx
Address: 10.250.b.b
~~~
<br>
<br>

## <u>3. Current DNS Serverの確認</u>

主催者側に質問をしてみたところ、Current DNS Serverが10.250.aaa.aaaになっているか確認するよう指示があり、確認してみる。

~~~
$ resolvectl status

Global
       Protocols: 
resolv.conf mode: 

Link 2 
    Current Scopes: 
         Protocols: 
Current DNS Server: 136.c.c.c
       DNS Servers: 136.c.c.c 10.250.a.a 8.8.8.8
~~~
<br>

Current DNS Serverが違っている。
ということで、これを10.250.a.aにしてみる。

<br>

## <u>4. Current DNS Serverの変更</u>

Chat-gpt先生に質問してみたところ、以下の手順を教えてくれた。


**1. `/etc/systemd/resolved.conf`を編集**

resolvectlが管理するDNS設定は`/etc/systemd/resolved.conf`で設定可能とのこと。

この`resolved.conf`の`DNS=`と`FallbackDNS=`を設定するよう指示あり。
#をコメントアウトして、値を入れる。

~~~
$ sudo vi /etc/systemd/resolved.conf

[Resolve]
DNS=10.250.a.a
FallbackDNS=8.8.8.8
~~~
<br>

**2.  systemd-resolvedサービスを再起動**

設定を反映させるために、`systemd-resolved`を再起動。

~~~
$ sudo systemctl restart systemd-resolved
~~~
<br>

**3. resolvectl statusで確認**

設定が反映されたかどうかを再度reslovectl statusコマンドで確認。

~~~
$ resolvectl status

Global
           Protocols: 
    resolv.conf mode: 
         DNS Servers: 10.250.a.a
Fallback DNS Servers: 8.8.8.8

Link 2 
Current Scopes: DNS
     Protocols: 
   DNS Servers: 136.c.c.c 10.250.a.a 8.8.8.8

~~~
<br>

Current DNS Serverの表示がなくなり、代わりにGlobal側でDNSの値が明示された。

<br>

**4. nslookup・curlで確認**

この状態でnslookupとcurlを実行してみたところ、特にDNSを指定しなくても疎通できるようになった！

<br>
