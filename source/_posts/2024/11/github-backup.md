---
title: GitHubでバックアップ
date: 2024-11-02 18:03:13
tags:
 - Hexo
 - Icarus
 - Git
category: blog
---



ブログ立ち上げは無事できたが、現状ではGitHubでホストできているのは公開用データのみで、記事データ（.mdファイル）などはバックアップできていない。

＃Hexoはフォルダ内のデータから`hexo generate`コマンドでPublicフォルダを生成し、指定した公開場所（_config.ymlの#Deployment）にデプロイする。

職場で日頃から「端末は一番大事な時に壊れたりするのでローカルだけで持ってはいけない、必ずクラウドで管理すること」と言われているので、ブログデータ全体をGitHubでバックアップしながらGitHub Pagesにデプロイする方法を検討。

<br>

---
# 手順
---

main(master)ブランチでバックアップを取りつつ、gh-pagesブランチにデプロイする。
<br>

- Hexoを入れているローカルフォルダをGit管理にする
- コミットする
- ローカルフォルダとGitHubのリモートリポジトリを紐づける
- プッシュする
- `_config.yml`を書き換えてデプロイする

<br>

まずはローカルフォルダをGit管理にする。

~~~
$ cd myblog
$ git init
~~~
<br>

コミットする。

~~~
$ git add .
$ git commit -m "XXXXX"
~~~
<br>

GitHubからリモートリポジトリのURLをコピーしてきたら、ローカルと紐づける。

~~~
$ git remote add origin https://github.com/XXXXX
~~~
<br>

プッシュする。

~~~
git push origin main
~~~

ローカルのメインリポジトリ名をmain、リモートがmasterになっていたせいでブランチが二つできてしまった。。。

GitHubでデフォルトのブランチをmasterからmainに変更して、masterブランチを削除。

その後、`_config.yml`の`branch:`をmasterからgh-pagesに編集。

~~~
deploy:
  type: git
  repo: git@github.com:XXXXXX.git【GitHubのリポジトリのURL】
  branch: gh-pages #ここをmasterから変更
~~~
<br>

これまでの作業を保存（add, commit , push）して`hexo deploy`すると、リモートにgh-pagesブランチが作成される。

あとはGitHubのPagesの設定（Build and deployment）の「Branch」をgh-pagesブランチに変更すればOK.

<br>
---

今後は編集するたびに以下のコマンドを繰り返せばよさそう。

~~~
$ hexo generate
$ git add
$ git commit -m "XXXXX"
$ git push origin main
$ hexo deploy
~~~
<br>

参考にさせていただいたサイト
https://qiita.com/nyu___nS/items/3fca57ce133be69835ba#comments

<br>
<br>
