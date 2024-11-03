---
title: GitHubのユーザ名を変更
date: 2024-11-02 22:59:52
tags:
 - Git
category: tech
---

雑につけたGitHubの名前を変えたくなり、まだ取り返しがつくうちに変更。

挙動がおかしくなるんじゃないかと不安になりつつ、↓の記事をみて多分大丈夫だと思い決行。

[適当に付けたGitHubのユーザー名を変えたくなった話](https://qiita.com/plant0322/items/c278ef6d42d096714aa9)

<br>

---
# **手順**
---

<!-- toc -->
<br>

### GitHubにログインして設定画面から変更

GitHubにログインして「Setting」→「Account」をクリックする。

表示された画面の「Change username」をクリックして、出てくるWarningっぽいポップアップの中の`I understand, let's change my username`を更にクリック。

新しいユーザ名を入れるウィンドウが出てくるので、ユーザ名を入力して「Change username」をクリックして完了。

<br>

### ローカルの.gitconfigファイルを編集

エディタで直接編集。

~~~
$ cd ~
$ subl .gitconfig

[user]
  name = XXXX
  email = XXXX
~~~

<br>

### Git管理のファイルの設定変更

現状はこのブログのファイルのみがGit管理下。まずリモートリポジトリを確認＆修正。

~~~
$ cd myblog
$ git remote -v
$ git remote set-url origin 【新しいURL】
~~~

<br>

`_config.yml`も修正。

~~~
url: https://【新しいユーザ名】.github.io/myblog/

deploy:
  type: git
  repo: 【新しいURL】
  branch: gh-pages
~~~



