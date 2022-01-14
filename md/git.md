# ロボットシステム学第3回

上田 隆一

千葉工業大学

---

## 今日の内容

* Git, GitHub

---

## 自分で書いたプログラムの管理

* バックアップ、バージョン管理をどうするか？
  * ラズパイやマイコンのコードの管理は結構面倒
    * 別にファイルやディレクトリのコピーして紛失
    * ファイルをhoge.c, hoge.c.org, hoge.c.orgorg...とコピーして煩雑に<br />　
* さらに便利にならないか？
  * バックアップ場所を決める
  * 変更の履歴を管理したい

---

## <span style="text-transform:none">Git</span>

* 版管理（バージョン管理）システム
  * ファイルの変更履歴を管理するためのシステム<br />　
* Linus Torvalds が作成
  * Linux の共同開発のため

---

## <span style="text-transform:none">Gitのインストール</span>

* やること
  * `sudo apt install git` 等
    * 最近はデフォルトで使える環境が多い
  * ユーザの設定

```bash
$ sudo apt install git
###自身の名前とe-mail アドレスを記録しておく###
$ git config --global user.name "Ryuichi Ueda"
$ git config --global user.email "ueda@hogehoge.com"
###エディタも登録しておくとよい###
$ git config --global core.editor vim
###確認###
$ cat .gitconfig
[user]
name = Ryuichi Ueda
email = ueda@hogehoge.com
[core]
editor = vim
```

---

## <span style="text-transform:none">GitHub</span>

* Gitを利用したサービス
  * 「リポジトリ」のホスティングと公開、コミュニケーション
      * リポジトリ: あるプロジェクトに関するファイルの集まり
  * 公開しないリポジトリも作成可能<br />　
* 利用方法
  * ウェブサイト
  * コマンドライン

---

## アカウント作成

* GitHubにアクセス
  * https://github.com/
* ユーザ名、email アドレス、パスワードを決めて"Sign up for GitHub"
  * ユーザ名は恥ずかしくないものを！
* 画面指示に従う
* プランを選ぶときに"Free"が選択されているのを確認して"Finish sign up"
* 登録したメールアドレスに確認メールが届くのでインストラクションに従う
* できる人は公開鍵の登録

---

## リポジトリの作成

* GitHubにリモートのものを一つ作ってみましょう
* GitHubのサイトでの操作
  * 右下あたりにある"New repository"をクリック
  * 必要事項を記入
    * 後の操作の例のために以下をお願いします
      * "Initialize this repository with a README"にチェック
      * ライセンスのプルダウンからMITライセンスを選択
  * "Create repository"ボタンを押す
* ウェブ画面にリポジトリの画面
  * READMEとLICENSEができている

---

## リモートのリポジトリをローカルに

* リポジトリの画面の"Code"をクリック
* 「Use HTTPS」を選択してURLをコピー
  * クリップボードのアイコンをクリックするとコピーできる
* リポジトリを作りたいディレクトリで次の操作

```bash
$ git clone <さっきクリップボードにコピーした文字列をペースト>
Cloning into 'test'...
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), done.
Checking connectivity... done.
$ cd test/
$ ls -a
.  ..  .git  LICENSE  README.md
```

---

## リポジトリにコードを追加

* なんでもいいからプログラムを一つ置く
  * 特にこだわりがなければ次のようなシェルスクリプトを
    ```bash
    $ cat hoge.bash
    #!/bin/bash
  
    echo hoge
    ```
* 「addしてstatus見てcommit」
  * リポジトリへのファイルの登録
    * まず、登録対象のファイルを選択（git add）
    * 選択されたファイルは「ステージ」に載せられる
      * 「ステージング」 と表現
      * git statusで確認
  * ステージに置いたファイルを登録（git commit）
    * 「コミットする」と表現

---

## コミットの操作

```bash
$ git add hoge.bash
$ git status
On branch main
Your branch is up-to-date with 'origin/main'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
new file:   hoge.bash
$ git commit -m "Add a file"
[master 9da0223] Add a file
 1 file changed, 3 insertions(+)
 create mode 100644 hoge.bash
```

---

## リモートへの「push」

* ローカルのコミットをリモートへ反映
  * origin: リモートのリポジトリのこと
  * main: mainブランチ

```bash
$ git push origin main
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 333 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/ryuichiueda/test.git
   b09eefb..9da0223  main -> main
```

---

## ブランチ

* ディレクトリの中の状態を分岐したもの
  * ブランチ = 枝
* 2020/10/01 からデフォルトのブランチ名が「master」から「main」に変更になりました
  ```bash
  ・$ git branch
  * main
  ```
* 開発用ブランチを作りましょう
  ```bash
  $ git checkout -b dev
  Switched to a new branch 'dev'
  $ git branch
  * dev
    main
  ```

---

## ブランチでの作業とマージ(1/2)

* 開発をdevブランチで進めてコミット
  * hogeを2個にする
  ```bash
  $ cat hoge.bash （hoge.bashを書き換えた後にcatしたところ）
  #!/bin/bash
  echo hoge
  echo hoge
  $ git add hoge.bash 
  $ git commit -m "Add another hoge"
  [dev ce996b6] Add another hoge
   1 file changed, 1 insertion(+), 1 deletion(-)
  ```
* mainブランチに戻ってみましょう
  ```bash
  $ git checkout main
  Switched to branch 'main'
  Your branch is up-to-date with 'origin/main'.
  $ git branch
    dev
  * main
  ```


---

## ブランチでの作業とマージ(2/2)
  
* hoge.bashが元に戻っている
  ```bash
  $ cat hoge.bash
  #!/bin/bash
  echo hoge
  ```
* devの成果をmainに取り込む
  ```bash
  $ git merge dev
  Updating 9da0223..ce996b6
  Fast-forward
  hoge.bash | 2 +-
  1 file changed, 1 insertion(+), 1 deletion(-)
  ```
* ついでにpushしておきましょう
  ```bash
  $ git push   #mainのpushは2回目以降、これで良い
  ```
* main以外のブランチもリモートにpush可能 

---

## 別のマシン/ディレクトリ/人とのリポジトリの共有(1/2)

* リポジトリは何個でも作れる
* 別のマシーンかディレクトリにクローンして、編集してpushしてみましょう
  ```bash
  $ cd /tmp/
  $ git clone https://github.com/ryuichiueda/test.git
  $ cd test
  $ cat hoge.bash 
  #!/bin/bash
  echo hoge
  echo hoge
  echo hoge      #もう一個増やす
  $ git add hoge.bash
  $ git commit -m "Third hoge"
  $ git push
  ```

---

## 別のマシン/ディレクトリ/人とのリポジトリの共有(2/2)

* 元のリポジトリに戻ってこの更新を反映してみましょう
  ```bash
  $ cd ~/GIT/test/  #ディレクトリは各自最初にcloneしたところを指定
  $ cat hoge.bash   #まだechoが二つ
  #!/bin/bash
  echo hoge
  echo hoge
  $ git pull
  remote: Counting objects: 3, done.
  remote: Compressing objects: 100% (1/1), done.
  remote: Total 3 (delta 1), reused 3 (delta 1), pack-reused 0
  Unpacking objects: 100% (3/3), done.
  From https://github.com/ryuichiueda/test
     ce996b6..ff549b7  main     -> origin/main
  Updating ce996b6..ff549b7
  Fast-forward
   hoge.bash | 1 +
   1 file changed, 1 insertion(+)
  $ cat hoge.bash    #3つに
  #!/bin/bash
  echo hoge
  echo hoge
  echo hoge
  ```

---

## 発展

* [一昨年の資料（12ページ〜）](http://www.slideshare.net/ryuichiueda/20159-55493670)
  * フォーク
    * 人のリポジトリを自分のところに持ってくる
  * プルリクエスト
    * リポジトリの持ち主に修正したものを取り込んでもらう
* 他のキーワード
  * コンフリクト
    * 二つのリポジトリで同じ箇所を別の書き方で修正

---

## 宿題

* 自分の書いたプログラムやレポートをGitHubで公開してみましょう
    * 公開してはいけないものについてはプライベートで<br />　
* その他、バージョン管理が必要なものはGitHubへ
    * 卒論はGitHubで管理のこと
        * Wordで書くとバイナリ（差分がとれない上にファイルが大きい）なのでGitHubと相性悪いです。TeX使いましょう。

