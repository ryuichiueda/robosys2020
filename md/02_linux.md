# ロボットシステム学第2回

上田 隆一

2019年9月27日@千葉工業大学



---

## 今日の内容

* ノートPCとラズパイを接続

---

## やること

* 下の写真のようにノートPCとラズパイを有線LANで直接接続
* 次のような接続を構成
  * ラズパイ -> 有線LAN -> ノートPC -> ノートPCのWiFi -> インターネット
  * つまりノートPCをルータにする
 
<img width="35%" src="https://pbs.twimg.com/media/DKZLU64U8AEdULI.jpg" />

---

## ノートPCのOS別設定方法

* Windows: https://blog.ueda.tech/?p=8694
  * 拙ブログですみません
* Mac: https://blog.ueda.tech/?p=8717
  * こちらも拙ブログですみません
* ラズパイのIPの調べ方
  * `$ nmap -sP <有線側のセグメント>`
* Linux: DHCPを立てるのが面倒なので割愛

---

## <span style="text-transform:none">Linuxの世界

* ほぼ全てテキストファイルでできている
  * 設定ファイル等
  * /etc下のファイルを見てみましょう
    * $ cat /etc/passwd
* どうやって見る？編集する？
  * エディタ
  * コマンド
* エディタとコマンドは使えないとかなりストレス
    * 自転車と一緒で使えると自然に使えるようになるので我慢して慣れられるかどうかで今後が変わる

---

## エディタ

* （CLI, command line interface）でテキストファイルを読み書きするもの
  * EmacsとVimがメジャー
    * 特にここでは説明しません
  * 練習コマンドがあるのでそれで練習を
    * Vimにはvimtutorというコマンドがあり、最初はこれをやるのが一番良い

---

## コマンド

* CLIからプログラムを起動する場合は字をシェルに打ち込むが、その字のこと
  * プログラム自身のことも指し、このスライドではプログラム自身のこと
* だいたいこの二種類
  * システム操作のためのコマンド
    * サービスを立ち上げたり止めたり
    * これはマニュアルを見たら覚えられる
  * フィルタコマンド
    * 標準入力から文字を受けて標準出力に加工した字を出すもの
    * 組み合わせて使う（マニュアルがあまりない）

---

## フィルタコマンド

* 多数: grep, find, wc, ...
  * /bin/下に存在
* とりあえず最初に覚えるもの
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
* **ところで、出力がザーッと出て使いにくくないか？**

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

## ファイルの操作

* ls, mv, rm, cp, mkdir, rmdir
  * 口頭で説明します

---

## <span style="text-transform:none">man

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
* `$ ping 8.8.8.8`
  * GoogleのDNSにパケットが届くか確認
  * インターネットに出られるか確認する時によく使用
* `$ ping www.yahoo.co.jp`
  * ドメイン名をIPに変換できるか（名前解決）を確認する時に使用
* `$ ping 192.168.1.254`
  * 192.168.1.254に届くか確認
  * （192.168.1.254はルーターっぽいIPアドレス）
* たまにセキュリティーのために先方がふさがっていることがある

---

## <span style="text-transform:none">ip, ifconfig

* ネットワークの設定の確認
* ifconfig: 古い確認方法
  ```bash
  $ ifconfig
  ```
* ip: 新しい方法
  ```bash
  $ ip addr show    #IPアドレス等の確認
  ```

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

## プログラムのコンパイル

* 次のようなC言語のファイルhoge.cを作ってみましょう
  * 端末のエディタがまだ使えない人はこの資料のコピペやscp等を駆使のこと
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

## <span style="text-transform:none">Raspbianのメンテナンス

* rpi-update: カーネル等のアップデート
```
pi@raspberrypi:~ $ sudo rpi-update
```
* apt update: パッケージリストの更新
```
pi@raspberrypi:~ $ sudo apt update
```
* apt upgrade: パッケージのアップグレード
```
pi@raspberrypi:~ $ sudo apt upgrade
```
* raspi-config: Raspberry Piの機能設定
```
pi@raspberrypi:~ $ sudo raspi-config
```
  * ロケールの設定
  * 言語環境の設定
  * 起動時にGUIの抑制
  * ssh, SPI等の機能をON/OFF

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
