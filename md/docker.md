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
      * プロセス同士が互いにメモリを覗けない仕組みと類似


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
* 気軽に立ち上げ、破棄可能
* オーバーヘッドが少ない
* 条件が揃えばDockerのコンテナは他の環境でも動く
  * 別のバージョンのディストリビューション
  * 別のディストリビューション
  * 別のOS（の上で動くLinuxカーネル）

---

## コンテナを使う

* 講師の作成したコンテナのイメージ
  * ROSの回のウェブカメラを使った実演とおなじもの
    * [講師のDocker Hubのアカウント](https://hub.docker.com/u/ryuichiueda)にある
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

## コンテナの停止と削除

* コンテナ: 今動いている隔離環境
* 既存のコンテナの確認と削除

```
$ sudo docker container list -a
CONTAINER ID    IMAGE                               COMMAND                  
CREATED         STATUS              PORTS                      NAMES
59b2685b9b37    ryuichiueda/ros-camera-server:pi4   "bash /root/exec.bash"   
29 minutes ago  Up 28 minutes       0.0.0.0:8080->8080/tcp     friendly_mcnulty
$ sudo docker container stop 59b2
59b2
$ sudo docker container rm 59b2
59b2
```

---

## イメージの削除

* イメージ: コンテナを作るときのデータ
  * インストールCDのようなもの
* 既存のイメージの確認と削除

```
$ sudo docker image list -a
REPOSITORY                      TAG                 IMAGE ID            
CREATED             SIZE
ryuichiueda/ros-camera-server   pi4                 92e1a37aeca8        
42 hours ago        1.74GB
ubuntu@ubuntu:~$ sudo docker image rm 92e1
Untagged: ryuichiueda/ros-camera-server:pi4
Untagged: ryuichiueda/ros-camera-server@sha256:ce7483a3081180b4289d6c98a108a5e73e481cf749bab648e82e24403dcbda43
（略）
```

---

## コンテナのイメージを作る

* 今ダウンロードしたものと同じものを作ってみましょう 
* 準備
  * 作業ディレクトリを作成
```
$ mkdir camera-server
$ cd camera-server/
```

---

## <span style="text-transform:none">Dockerfile</span>

* `Dockerfile`: イメージを作る手順書
  * `make`における`Makefile`のようなもの
* 最初の`Dockerfile`
```
$ cat Dockerfile
FROM ryuichiueda/ubuntu18.04-pi4-ros-image
```
  * 元になるイメージを持ってくる
* ビルドのコマンドが長いので`Makefile`も作っておきましょう
  * インデントはタブで
  * `-t`でイメージの名前を指定
```
$ cat Makefile
build:
        sudo docker image build . -t camera-server
```

---

## ビルド

* `ryuichiueda/ubuntu18.04-pi4-ros-image`と、そのベースになっているレイヤーがダウンロードされてビルドされる
  * イメージはレイヤーが何層も重なったもの
    * OverlayFSによって実現

<span style="font-size:70%">

```
$ make
sudo docker image build . -t camera-server
Sending build context to Docker daemon  3.072kB
Step 1/1 : FROM ryuichiueda/ubuntu18.04-pi4-ros-image
latest: Pulling from ryuichiueda/ubuntu18.04-pi4-ros-image
04da93b342eb: Extracting [==================================================>]  23.73MB/23.73MB
b235194751de: Download complete
606a67bb8db9: Download complete
72d199f462f5: Download complete
6ce64c3ad07d: Download complete
8a61eb997224: Downloading [==============================>                    ]  26.61MB/43.53MB
f7c7d079b6f7: Download complete
04e8904462b3: Downloading [=>                                                 ]  10.24MB/355.4MB
0cfd5c654b33: Download complete
912911139a1f: Waiting
0c42652750fc: Waiting
```

</span>

---

## ROSパッケージのインストール

* セットアップのためのコマンドは`RUN`という命令で実行
  * `Dockerfile`に書いて`make`するとダウンロードとインストールが始まる

```
$ cat Dockerfile
FROM ryuichiueda/ubuntu18.04-pi4-ros-image

RUN apt-get update && apt-get install -y \
  ros-melodic-cv-camera \
  ros-melodic-web-video-server
```

* 18.04のイメージを使っているのでROSのバージョンはmelodic
  * ホストがUbuntu 20.04の場合、環境を18.04で作って20.04上で動かすことになる
    * Linuxのバイナリの互換性のおかげでバージョン、ディストリビューション違いがあっても動く場合が多い

---

## 起動スクリプトの準備

* ノードを立ち上げるスクリプトをリポジトリに置く
  * `rosrun`よりも`roslaunch`を使ったほうがよいかもしれないが場合による

```
$ cat run.bash
#!/bin/bash

source /opt/ros/melodic/setup.bash

roscore &
sleep 5

rosrun cv_camera cv_camera_node 2> /dev/null &
rosrun web_video_server web_video_server
```

---

## 起動スクリプトの設置


* `Dockerfile`でスクリプトをイメージ内に移設、設定
  * `COPY`: 外からイメージにファイルをコピー
  * `CMD`: コンテナを実行すると走り出すプログラムを設定 
  * 普通にコマンドを書いてもいいが、例ではリストを使って記述

```
$ cat Dockerfile
・・・
COPY ["run.bash", "/root/run.bash"]
RUN ["chmod", "+x", "/root/run.bash"]
CMD ["/root/run.bash"]
```

---

## 起動

* コマンドが長いので`Makefile`に書いてしまいましょう
  * `make run`で起動
    * シェルスクリプトに書くほうがよいかもしれない
  * すべてのノードが立ち上がったらブラウザでカメラの映像を確認

```
$ cat Makefile
・・・
run:
	sudo docker container run -p 8080:8080 --device=/dev/video0:/dev/video0 -t camera-server
```

* ポイント
  * コンテナ内からはデバイスファイルもポートも見えないので、引数で見えるように設定
    * 上の例: 8080番と`/dev/video0`

---

## 停止と再起動

* 停止: [9ページ](/lesson12_docker.html#/8)
```
$ sudo docker container list
CONTAINER ID        IMAGE               COMMAND             CREATED           ...
d6134762c95b        camera-server       "/root/run.bash"    29 minutes ago    ...
$ sudo docker container stop d613
d613
```
* （再）起動: `docker container start`を使用
```
$ sudo docker container start d613
d613
```

---

## <span style="text-transform:none">Docker Hubにリポジトリをpush</span>

* 手順だけ示します
  * 1. Docker Hubにサインアップ
  * 2. `docker login`
```
$ sudo docker login
Login with your Docker ID to push and pull images （略）
Username: ryuichiueda
Password:
（略）
Login Succeeded
```
  * 3. イメージの名前をきっちりつけてビルド
    * 例: `sudo docker image build . -t ryuichiueda/camera-server:pi4`
    * （`docker tag`で既存のものに名前をつけてもよい）
  * 4. `docker push`
```
$ sudo docker push ryuichiueda/ros-camera-server:pi4
```

---

## まとめ

* コンテナ: アプリケーションを隔離した環境
* Docker: 「コンテナ」を作ったり使ったりするためのソフトウェア/プラットフォーム
  * 「プロセスを仮想メモリ空間に隔離」からさらに1段階抽象化が起こっている<br />　
* 参考文献
  * [Docker-docs-ja](https://docs.docker.jp/index.html)
  * 山田明憲: Docker/Kubernetes 実践コンテナ開発入門, 技術評論社, 2018.
