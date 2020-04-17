# ロボットシステム学第2回

上田 隆一

千葉工業大学

---

## 今日の内容

* ノートPCとラズパイを接続

---

## やること

* 下の写真のようにノートPCとラズパイを有線LANで直接接続
* 次のような接続を構成
  * ラズパイ -> 有線LAN -> ノートPC -> ノートPCのWiFi -> インターネット
  * つまりノートPCをルータにする
 
<img width="35%" src="./md/images/raspi4_connect.jpeg" />

---

## ノートPCのOS別設定方法

* [Windows](https://twitter.com/ryuichiueda/status/1250644188992962560)
* [Mac](https://b.ueda.tech/?post=08717)
  * 拙ブログですみません
* ラズパイのIPの調べ方
  * `$ nmap -sP <有線側のセグメント>`
  * `$ arp -a
* Linux: DHCPを立てるのが面倒なので割愛

---

## ログインしてみましょう

* MacやLinuxならTerminal、WindowsならWSLを開く
    * `$`の右側に「コマンド」を打って操作
* ログインするためのコマンド:
```
$ ssh <ユーザ名>@<IPアドレス>
```
    * 具体例: `ssh ubuntu@192.168.137.3`
    * 意味
        * `ssh`: 暗号化して他の計算機（ホスト）と通信するためのコマンド
        * `ubuntu`: ログインするときの「ユーザ名」
            * ログインする前のPC側でのユーザ名を探してみましょう
        * IPアドレス: 接続先の住所


---

## 最初にやること

* ファイルの確認
  * 設定ファイル置き場である/etc下のファイルを見てみましょう
    * `ls /etc/`, `cat /etc/passwd`, `tree /etc/`などを使用
* プロセスの確認
  * `ps`, `top`, `pstree`などを使用

```
ubuntu@ubuntu:~$ pstree
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─2*[agetty]
（中略）
        ├─systemd-timesyn───{systemd-timesyn}
        ├─systemd-udevd
        ├─unattended-upgr───{unattended-upgr}
        └─wpa_supplicant
```

---

## <span style="text-transform:none">Linuxの</span>世界

* 「Linux（やUnix系OS）の世界」はとても単純
    * 2大重要事項
        * データは全て「<span style="color:red">ファイル</span>」（テキストファイル）で保存される
        * プログラムは「<span style="color:red">プロセス</span>」単位で動く
    * その他重要なこと
        * ファイルもプロセスも<span style="color:red">木構造</span>で管理されている
        * プロセスは<span style="color:red">コマンド</span>を手で打ったり設定ファイルに書いたりして起動

この基本構造でなぜかロボットが動く

---

## なぜか<span style="text-transform:none">Linux</span>でロボットが動く

* 一番良い例が一番身近にある
    * ロボットの行き先の決定、画像処理、モータへの指令、通信
    * ソフトウェア開発もLinuxで（GPUを使った深層学習など）
* 単純な仕組みで様々な機能を実現。守備範囲が広い。

<iframe width="560" height="315" src="https://www.youtube.com/embed/q1FH93icuHk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

## <span style="text-transform:none">Linux</span>でロボットの機能を自由自在に作るには

* 二大重要事項
    * 構造を理解する
    * 操作に慣れる（自転車と一緒）<br >　
* 習得の際に気をつけること
    * <span style="color:red">できることが増えないと面白さが分からない</span>
       * ほとんどの人にとっては、最初ははっきり言ってつまらない。
       * 教員を含めて楽しそうにしている連中に殺意しかわかない。
       * 教員が楽しさを演出しても裏目に出るだけ。

以後の時間は、みなさんを監禁して練習します。（面白くないので）

---

## ファイルの操作

* 生成、移動、削除、プログラムの実行・・・
    * /bin/などの下に存在するプログラム（ファイル）
    * コマンドを使う
        * ls, rm, cd, mkdir, rmdir, grep, find, wc, ...<br />　
* 例題
    * `aaa`というファイルと`bbb`を作り、`bbb`に`aaa`を入れる。その後、`aaa`を削除して`bbb`を削除。

---

## コマンドの練習

* cat: 表示
* sudo: root権限でコマンドを実行
  ```bash
  $ sudo cat /etc/shadow
  ```
* grep: 検索
  ```bash
  $ grep ueda /etc/passwd
  ueda:x:1000:1000:Ryuichi UEDA,,,:/home/ueda:/bin/bash
  ```
* find: ファイルの列挙
  ```bash
  $ find /
    ```

catやfindの結果をgrepしたり、findの結果を止めて眺めたりということがしたくなる

---

## コマンドの組み合わせ

* 特定のファイルを探すには？
  * 例: passwdファイルってどこにあったけ？
    * grepと組み合わせる
      ```bash
      $ sudo find / | grep passwd
       [sudo] password for ueda:
       /var/lib/dpkg/info/passwd.postinst
       /var/lib/dpkg/info/base-passwd.postinst
       /var/lib/dpkg/info/passwd.preinst
      ...
      ```
    * さらに絞り込む（grepで正規表現を使う）
      ```bash
      $ sudo find / | grep '/passwd$'
      /var/tmp/etc/init.d/passwd
      /var/tmp/etc/cron.daily/passwd
      ...
      ```
    * lessで見る
      ```bash
      $ sudo find / | less
      ```

---

## 正規表現の例

*  /etc/servicesの調査
  ```bash
  $ cat /etc/services | grep 80　　         #80を検索
  $ cat /etc/services | grep '[^0-9]80/'  #80番ポートのレコードだけ検索
  $ cat /etc/services | grep ftp                #ftpという語句を検索
  $ cat /etc/services | grep ^ftp       #最初がftpで始まる行を検索
  ```
* 上級者向け
  * AWKとgrepで10000番ポート以上のレコードを抽出のこと

---

## <span style="text-transform:none">man</span>

* マニュアルコマンド
  ```bash
  $ man grep
  ```
* 章がある
  ```bash
  $ man 1 printf      #1章（コマンド）のprintf
  $ man 3 printf      #3章（C言語の関数）のprintf
  ```
  * 詳しくは `$ man man` で。

---

## <span style="text-transform:none">apt, apt-get

* APT (Advanced Packaging Tool): Debian系Linuxのパッケージシステム
  * ソフトウェアのインストールに利用
  * 使用例
    ```bash
    $ sudo apt-get install nmap   #スクリプト用
    $ sudo apt install nmap         #手打ち用
    ```
* 最近は`snap`というのも出てきた

---

## <span style="text-transform:none">ping

* 通信先にパケットが届くか確認する時によく使う
* 例1: 
```
$ ping 8.8.8.8
```
  * GoogleのDNSにパケットが届くか確認
  * インターネットに出られるか確認する時によく使用
* 例2: 
```
$ ping www.yahoo.co.jp
```
  * ドメイン名をIPに変換できるか（名前解決）を確認する時に使用

---

## <span style="text-transform:none">ip

* ネットワークの設定の確認
  ```bash
  $ ip addr show    #IPアドレス等の確認
  $ ip a            #略記
  ```
* 問題: IPアドレスの行を抽出してみましょう
    * grepと組み合わせて

---

## その他通信関係のコマンド

* nmap: ネットワークのスキャン
  * インターネットに向けてやると最悪裁判になるので注意
  * 使用例
    ```bash
    $ nmap -sP 192.168.3.0/24   #自分のネットワークのマシンのIPアドレスを調査
    $ nmap 192.168.3.2 #開いているポートの調査
    ```
* traceroute
  * パケットの運ばれる経路を調査
  ```bash
  $ traceroute 8.8.8.8
  ```

---

## <span style="text-transform:none">scp, rsync

* マシン間でファイルをコピー
  ```bash
  $ scp hoge 192.168.3.1~/     #hogeファイルを192..1のホームへ
  $ scp 192.168.3.1:~/hoge2 ./  #逆向き
  $ rsync -av hoge/ 192.168.3.1:~/hoge/ #hogeディレクトリを192..1のホームへ
  $ rsync -av192.168.3.1:~/hoge/ hoge/   #逆向き
  ```
* TeraTermにもscp機能がある



---

## 停止・再起動

* 停止
  * 次の2通り
  ```bash
  $ sudo shutdown -h now
  $ sudo poweroff
  ```
  * Linuxはデータをストレージに後から書き込むので、ラズパイの場合は緑のランプが点滅しなくなるまでよく待つ
    * ストレージ: HDD, SSD, microSD, ...
    * （ラズパイはプログラムやデータ等を別のマシンで管理する場合が多いのでブチ切りする人も多い）
* 再起動
  ```bash
  $ sudo reboot
  ```

---

## ファイルの編集

* 「エディタ」を使う
    * ここではCLI（command line interface）でテキストファイルを読み書きするものを指す
        * EmacsとVimがメジャー
        * 特にここでは説明しません
    * 設定ファイルやプログラム、文章までなんでもこれで書く
    * 練習コマンドがあるのでそれで練習を
        * Vimにはvimtutorというコマンドがあり、最初はこれをやるのが一番良い


---

## プログラムのコンパイル

* 次のようなC言語のファイルhoge.cを作ってみましょう
  ```c
  #include <stdio.h>
  int main(int argc, char const* argv[]){
      puts("あばばばば");
      return 0;
  }
  ```
  * gccを使ってコンパイルし、a.outを実行
  ```c
  $ gcc hoge.c
  $ ./a.out
  もしa.outが実行できなかったら $ chmod +x a.out
  ```

---

## スクリプトの実行

* 次のようなPythonのファイルhoge.pyを作ってみましょう
  ```python
  #!/usr/bin/python
  print "Java"
  ```
  * `#!`: シバン（shebang）
    * どのプログラムでこのスクリプトを実行するか指定
* 以下を実行
  ```bash
  $ chmod +x hoge.py
  $ ./hoge.py
  ```

---

## 次回までの宿題

* エディタが苦手な人はvimtutorをやっておく
  `$ vimtutor`
* 得意な人はVimやEmacsにプラグインを仕込む
  * Vimの場合はdein等
    * https://github.com/Shougo/dein.vim

---

## 補足

* SSH接続でこういうのが出た時の対処
  ```bash
  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
  @ WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! @
  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
  ```
  * → `~/.ssh/known_hosts` から接続先のIPアドレスを見つけてレコードを削除
