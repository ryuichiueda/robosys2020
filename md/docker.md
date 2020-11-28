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
  * ROSの回のウェブカメラを使った実演とおなじもの
    * ラズパイで動作
    * ノートPCで動かす場合は`:pi4`というタグを除去して実行
  * 手元のブラウザでラズパイのカメラ映像が見える

<span style="font-size:60%">

```
$ sudo apt install docker.io                       <-dockerのインストール
### いきなり実行 ###
$ sudo docker container run -p 8080:8080 --device=/dev/video0:/dev/video0 -t ryuichiueda/ros-camera-server:pi4
Unable to find image 'ryuichiueda/ros-camera-server:pi4' locally
pi4: Pulling from ryuichiueda/ros-camera-server
04da93b342eb: Pull complete
（略）
8a61eb997224: Extracting [===========================================>       ]  37.62MB/43.53MB
f7c7d079b6f7: Download complete
04e8904462b3: Downloading [================================>                  ]  227.7MB/355.4MB
（略）
started roslaunch server http://59b2685b9b37:44911/        <- ROSマスタが立ち上がる
ros_comm version 1.14.10
（略）
[ INFO] [1606550046.573381331]: Waiting For connections on 0.0.0.0:8080  <-webサーバが立ち上がる
```

</span>

---

## コンテナのイメージを作る

* 
