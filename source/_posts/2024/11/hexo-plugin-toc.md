---
title: Hexoに目次のプラグインを導入
date: 2024-11-03 10:26:31
tags: 
 - Hexo
 - Icarus
 - Git
category: blog
---

記事レイアウトの微調整。tocというプラグインで目次の自動生成ができるとのこと。

<br>

---
# **手順**
---

<!-- toc -->


### プラグイン（hexo-toc）のインストール

~~~zsh
$ npm install hexo-toc --save
~~~
<br>

### configファイルを編集

_config.icarus.ymlに以下を追記。

~~~yml
toc:
  maxdepth: 3
  class: toc
  slugify: uslug
  anchor:
    position: after
    symbol: "#"
    style: header-anchor
~~~
<br>

### 記事内にタグを設置

記事内の目次を表示したい場所に以下のタグを設置。見出しから自動的に目次を出力してくれる。

~~~.md
<!-- toc -->
~~~
<br>

参考にさせていただいた記事は以下。
[HEXO の投稿に目次機能を追加する](https://fennote.fareastnoise.com/2022/03/04/toc/)


<br>
<br>