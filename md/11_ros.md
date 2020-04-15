# ロボットシステム学第13回

上田 隆一

2019年12月6日@千葉工業大学

---

## 今日の内容

* ROS

---

## <span style="text-transform:none">ROS: robot operating system

* ロボットのソフトウェアコンポーネントを作って
  動作させるためのフレームワーク/ミドルウェア
  * OSでは無い
* 発祥: 2000年代後半、Willow Garage社
* BSDライセンス
* Linux（Ubuntu）上での動作がデフォルト
* サイト
  * 公式ページ: http://www.ros.org/
  * マニュアル等: http://wiki.ros.org/ja


---

## どんなものか

* 本体: プロセス間通信をつかさどる
  * プロセス同士をpublish-subscribeモデルやサービスでつなぐ
  * XML-RPC等を利用
  * 通信するデータに型
* 周辺
  * ビルドシステム、パッケージ管理、テストツール、・・・


<span style="color:red;font-size:50%">と書いてもよくわからんのでこちらで動かしてみます</span>



---

## デモ

* カメラの画像をブラウザから見る
* 手順
  * 1: ROS・ワークスペースのセットアップ（来週）
  * 2: 必要なパッケージのダウンロードとビルド
    ```bash
    $ sudo apt install ros-melodic-cv-camera
    $ sudo apt install ros-melodic-cv-bridge
    $ sudo apt install ros-melodic-web-video-server
    ```
  * 3: 見る
    ```
    $ roscore &
    $ rosrun cv_camera cv_camera_node &
    $ rosrun web_video_server web_video_server
    ブラウザでhttp://<IPアドレス>:8080/stream?topic=/cv_camera/image_raw
    ```

---

## さらに便利に使う

* ROS化されている重要ソフトウェア
  * gmapping, cartographer, ナビゲーションメタパッケージ
    * 地図生成（次のページにデモ）、位置推定、経路生成
  * MoveIt!
    * 腕の動作計画 腕先の位置を入力→関節角を計算（逆運動学）
  * その他、様々なハード・ソフトがROS化

---

## ROSを使ったSLAMの応用例

* コントローラでロボットに移動経路を教え、<br />そのあとロボットが自律で経路を移動
    * 移動経路を教えているときにロボットは<br />SLAMで地図を作り、通った位置を記録

<iframe width="560" height="315" src="https://www.youtube.com/embed/eVHkHOCsHns" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

* ポイント: SLAMのコードは一切書いてない

---

## ROSを使ったマニピュレーションの様子


* https://twitter.com/i/status/1201399538541400064
  * 2年生有志作
  * 動きはMoveIt!が生成

---

## ROSのインストール

* Ubuntu 18.04にインストールして使用
  * [GitHubのリポジトリ](https://github.com/ryuichiueda/ros_setup_scripts_Ubuntu18.04_server)にインストーラ
    * 中にあるシェルスクリプトをstep0.bash, step1.bash, locale.ja.bashと実行すればOK<br />　
* 講義ではインストール済みのイメージファイルを利用
  * [このページ](https://b.ueda.tech/?post=20190618_raspimouse)の「Raspberry Pi 3B+用のUbuntu 18.04にROSをインストールしたもの」

---

## 動作確認

* `roscore`
  * ROSの基盤となるプログラムが立ち上がる
  * Ctrl+cで出る

```bash
$ roscore
（略）
started roslaunch server http://localhost:39310/
ros_comm version 1.12.6

SUMMARY
========

PARAMETERS
* /rosdistro: melodic
* /rosversion: 1.12.6

NODES

auto-starting new master
process[master]: started with pid [1439]
ROS_MASTER_URI=http://localhost:11311/

setting /run_id to b749a100-d0dc-11e5-a506-b827eb17cb96
process[rosout-1]: started with pid [1452]
started core service [/rosout]
```

---

## ワークスペースの準備

* ワークスペース: 作業場
  * 構築手順
```bash
$ cd
$ mkdir -p catkin_ws/src
$ cd ~/catkin_ws/src
$ catkin_init_workspace 
Creating symlink "/home/ubuntu/catkin_ws/src/CMakeLists.txt" pointing to "/opt/ros/melodic/share/catkin/cmake/toplevel.cmake"
$ ls
CMakeLists.txt
```
  * .bashrcの末尾に以下を記述
```bash
source /opt/ros/melodic/setup.bash          #これは元からある
source ~/catkin_ws/devel/setup.bash         #ここから3行追加
export ROS_MASTER_URI=http://localhost:11311
export ROS_HOSTNAME=localhost
```

---

* 構築手順の続き
  * 環境のビルド
```bash
$ cd ~/catkin_ws
$ catkin_make
$ source ~/.bashrc
```
  * 確認
    * ROS_PACKAGE_PATHにcatkin_ws/srcがセットされているはず
```bash
$ echo $ROS_PACKAGE_PATH
/home/ubuntu/catkin_ws/src:/opt/ros/melodic/share
```

---

## ROSのノード

* プログラムのプロセス一つ一つが「ノード」と呼ばれる
* ノードの使用例（seiga-k/sysmon_rosパッケージを使う）
  * ハードウェアの状態をモニタするROSパッケージをインストール
```
$ sudo apt install ros-melodic-roslint 
$ cd ~/catkin_ws/src
$ git clone https://github.com/seiga-k/sysmon_ros.git
$ cd ..
$ catkin_make -j 1
$ roscore &
$ roslaunch sysmon_ros sysmon.launch 
```
  * ノードの確認
```
### ディレクトリのように管理されている ###
$ rosnode list
/rosout
/sysmon/cpumon
/sysmon/diskmon
/sysmon/memmon
/sysmon/netmon
/sysmon/tempmon
```

---

## トピック・メッセージ

* 今度はrostopic listと打ってみる
  * データをやり取りする口（トピック）が表示される
```
$ rostopic list
/rosout
/rosout_agg
/sysmon/cpumon/cpu
/sysmon/cpumon/cpu0
/sysmon/cpumon/cpu1
・・・
```
* トピックからデータを取り出す
  * このデータは「メッセージ」と呼ばれる
```
$ rostopic echo /sysmon/netmon/eth0
if_name: "eth0"
ip: [192.168.2.20]
tx_bps: 0
rx_bps: 4432
tx_error_rate: 0.0
rx_error_rate: 0.0
・・・
```

---

## パブリッシャ・サブスクライバ

* 各ノードがトピックを通じてメッセージを融通することで全体として仕事を行う
* データを出す側が**パブリッシャ**
* データを受け取る側が**サブスクライバ**
* この構造でノードの柔軟な組み換えが可能に
  * インタフェースが同じならプログラムを取り替えられる
  * 例: mjpeg_server（古い） -> web_video_server（新しい）

---

## ROSプログラミングの準備

* パブリッシャ、サブスクライバを作ってみましょう

---

## パッケージを作る

* パッケージ: いくつかのノードを含んだ一単位
* パッケージの生成
  * `catkin_create_pkg <作るパッケージの名前> [使用するライブラリ...]`
  * `rospy`: Pythonでノードを作るときに使用
  * パッケージを作ったら下に`scripts`というディレクトリを作成
    * ここにノードとなるプログラムを置く

```bash
$ cd ~/catkin_ws/src
$ catkin_create_pkg mypkg rospy
 Created file mypkg/package.xml
 Created file mypkg/CMakeLists.txt
 Created folder mypkg/src
 Successfully created files in /home/ubuntu/catkin_ws/src/mypkg. Please adjust the values in package.xml.
$ cd mypkg/
$ mkdir scripts
$ cd scripts/
```

---

## パブリッシャを作る

* 次のようなプログラム（count.py）を書いてみましょう
* ノード名が「count」、パブリッシャが「count_up」
* rospy.Publisherを作って定期的にデータを投げる
  * count_upというトピックに、型はInt32で（バッファとなるキューのサイズは1）

```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import Int32

rospy.init_node('count')
pub = rospy.Publisher('count_up', Int32, queue_size=1)
rate = rospy.Rate(10)
n = 0
while not rospy.is_shutdown():
    n += 1
    pub.publish(n)
    rate.sleep()
```

---

## ノードの実行

注意: あらかじめroscoreを立ち上げておきましょう。

```bash
$ rosrun mypkg count.py
```

* rosnode listとrostopic listでノードとトピックの確認を
* 次にrostopic echoでcount_upからデータを取り出してみましょう

```
$ rostopic echo /count_up 
data: 1430
---
data: 1431
---
...
```

---

## サブスクライバを作る

* 次のようなtwice.pyを作る
* rospy.Subscriberを使う
  * count_upという名前のトピックを購読する
  * 型はInt32
  * データを受け取ったときにcbという関数で処理
    * コールバック関数

```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import Int32

def cb(message):
    rospy.loginfo(message.data*2)

if __name__ == '__main__':
    rospy.init_node('twice')
    sub = rospy.Subscriber('count_up', Int32, cb)
    rospy.spin()
```

---

## <span style="text-transform:none">twice.pyの実行

* roscore、rosrun mypkg count.pyを事前に実行しておく
* 二倍になった数字が端末上に表示される
  * 非同期なので欠ける可能性があることに注意
  * 途中で切ってまた立ち上げても再開

```bash
$ rosrun mypkg twice.py
[INFO] [1484659840.762425]: 4
[INFO] [1484659840.862150]: 6
[INFO] [1484659840.961791]: 8
[INFO] [1484659841.062162]: 10
...
```

---

## パブリッシャとサブスクライバの同居

* twice.pyにパブリッシャを追加

```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import Int32

n = 0

def cb(message):
    global n
    n = message.data*2

if __name__ == '__main__': 
    rospy.init_node('twice')
    sub = rospy.Subscriber('count_up', Int32, cb) 
    pub = rospy.Publisher('twice', Int32, queue_size=1) 
    rate = rospy.Rate(10)
    while not rospy.is_shutdown():
        pub.publish(n)
        rate.sleep()
```

---

## 実行

* ノードを立ち上げて`rostopic echo /twice`でトピックとしてデータを得る

```bash
$ rostopic echo /twice
data: 1050
---
data: 1052
---
data: 1054
---
data: 1056
...
```

---

## その他（1/2）

* 型
  * `std_msgs`で定義されているものの他にもたくさん
    * `rosmsg list`を打ってみましょう
  * 自分で定義することも可能
* サービス
  * あるノードが他のノードのプログラムを実行
  * トピックと異なり、処理が終わるまでウェイトがかかる（同期処理）
* actionlib
  * サービスと同じくノード間でプログラムを呼び出す仕組み
    * 中断したり途中経過をみたりすることが可能
    * 時間のかかる処理の実装に利用される


---

## その他（2/2）

* package.xml
  * パッケージの情報が書かれる
  * メンテナの名前、ライセンス、他のパッケージの依存関係等、配布に必要な情報
* CMakeLists.txt
  * ビルド情報
  * 自分で型を作る、C++でノードを作る等、講義の内容より複雑なことをするときには手を入れる必要がある
* 参考
  * [小倉: ROSではじめるロボットプログラミング, 工学社, 2015.](https://www.kohgakusha.co.jp/books/detail/978-4-7775-1901-9)
  * [上田: Raspberry Piで学ぶ　ROSロボット入門, 日経BP, 2017.](http://ec.nikkeibp.co.jp/item/books/261040.html)
