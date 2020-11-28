# ロボットシステム学第12回

上田 隆一

千葉工業大学

---

## 今日の内容

* Dockerを使ってみる
* Dockerのイメージを作ってみる


---

## <span style="text-transform:none">Docker</span>

* 「コンテナ」を作ったり使ったりするためのソフトウェアであり、プラットフォーム<br />　
* コンテナ
  * 隔離環境にソフトウェアを詰め込んだもの
      * cgroup、OverlayFSやネームスペースの分離というLinuxの機能を駆使して隔離
  * コンテナの中ではソフトウェアが自由に動けるが、コンテナの外は見えない
      * プロセス同士が仮想メモリの仕組みで互いにメモリを覗けない仕組みと類似


---

## プロセスの隔離

* プロセスのネームスペースを分けて外を見えなくする
```
ubuntu@ubuntu:~$ sudo unshare --fork --pid --mount-proc bash
root@ubuntu:/home/ubuntu# top
（略）
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0    7892   3448   2940 S   0.0   0.1   0:00.01 bash
      9 root      20   0    9852   3168   2740 R   0.0   0.1   0:00.03 top -c
### ↑topを打ってもほとんどプロセスが出てこない ###
```
  * 出典: https://twitter.com/satoru_takeuchi/status/1273569132139606017 <br />　

---

## ファイルシステムの隔離

* Dockerで使われる仕組みとよく似たchrootの例
```bash
$ mkdir ~/tmp
$ cd ~/tmp
$ rsync -av /bin/ ./bin/      <-コマンドをコピーしておく
$ rsync -av /lib/ ./lib/      <-コマンドが使うライブラリもコピー
$ sudo chroot ./              <-chroot実行
bash-5.0# ls /
bin  lib                      <- ~/tmpがルートになっている
bash-5.0# exit
exit
$ ls /
bin   dev  home  lost+found  mnt  proc  run   snap  sys  usr
boot  etc  lib   media       opt  root  sbin  srv   tmp  var
```


---

## コンテナの利用例

* こんなふうに使える
  * ウェブサイトをウェブサーバーごとコンテナに閉じ込めて、通信のポートだけ開けて使用
    * あとで似たようなものを作ります
  * シェル芸bot
  * TeXのソフト一式をコンテナに閉じ込めて、外から機能を利用
 
特にコンテナでなくても良さそうだがなぜ？

---

## 何が嬉しいか

* コンテナのイメージをダウンロードしたらすぐ使える
* ホスト環境にいろいろインストールしなくてよい
* 隔離の方法が仮想マシンより効率的
* 条件が揃えばDockerのコンテナは他の環境でも動く
  * 別のバージョンのディストリビューション
  * 別のディストリビューション
  * 別のOS

---

## コンテナを使う

* 講師の作成したコンテナのイメージ
  * ROSの回のウェブカメラを使ったもの
    * ラズパイでは動かないのでノートPCにLinux入っている人だけ

```
$ sudo apt install docker.io                       <-dockerのインストール
$ sudo docker pull ryuichiueda/ros-camera-server   <-コンテナのイメージのダウンロード
Using default tag: latest
latest: Pulling from ryuichiueda/ros-camera-server
5b7339215d1d: Pull complete         <-左側の16進数は「レイヤ」のID
（略）
0cfdd66c6555: Downloading  77.89MB/176.7MB
3fffddebd6dc: Download complete
f3127908e59a: Download complete
### 実行 ###
$ sudo docker container run -p 8080:8080 --device=/dev/video0:/dev/video0 -t ryuichiueda/ros-camera-server
```


---

## コンテナのイメージを作る

* 
