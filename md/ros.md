# ロボットシステム学第10回

上田 隆一

---

## 今日の内容

* ROS

---

## <span style="text-transform:none">ROS: robot operating system

* ロボットのソフトウェアコンポーネントを作って動作させるためのフレームワーク/ミドルウェア
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
  * プロセス同士をpublish-subscribeモデルやclient-serverモデルでつなぐ
  * XML-RPC等を利用
  * 通信するデータに型<br />　
* 周辺
  * ビルドシステム、パッケージ管理、テストツール、・・・

<span style="color:red;font-size:70%">と書いてもよくわからんので使うメリットから</span>

---

## ROS化されている<br />重要ソフトウェア
  * gmapping, cartographer, ナビゲーションメタパッケージ
    * 地図生成（次のページにデモ）、位置推定、経路生成<br />　
  * MoveIt!
    * 腕の動作計画 腕先の位置を入力→関節角を計算（逆運動学）<br />　
  * 各種センサのインタフェース
    * すぐ使える
    * 以前は（特にLinuxでは）自分でシリアル通信のプログラムを書くなどの苦労があった

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
    * 自分で計算しなくていい（教員苦笑い）

---

## ROSのインストール

* Ubuntu 20.04にインストールして使用
  * [GitHubのリポジトリ](https://github.com/ryuichiueda/ros_setup_scripts_Ubuntu20.04_server)にインストーラ
    * 中にあるシェルスクリプト`step0.bash`、`step1.bash`を実行すればOK
* `roscore`と打つ
  * ROSの基盤となるプログラムが立ち上がる
  * Ctrl+cで出る

```bash
$ roscore
（略）
started core service [/rosout]
（立ち上がりっぱなしに）
```

---

## ワークスペースの準備

* ワークスペース: 作業場
* 構築手順
  * ディレクトリの作成
```bash
$ cd
$ mkdir -p catkin_ws/src
$ cd ~/catkin_ws/src
$ catkin_init_workspace 
Creating symlink "/home/ubuntu/catkin_ws/src/CMakeLists.txt" pointing to 
"/opt/ros/melodic/share/catkin/cmake/toplevel.cmake"
$ ls
CMakeLists.txt
```
  * .bashrcの末尾に以下を記述
```bash
source /opt/ros/noetic/setup.bash          #これは元からある
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
/home/ubuntu/catkin_ws/src:/opt/ros/noetic/share
```

---

## ROSパッケージ

* ROS用のソフトウェアパッケージ
* 利用方法
  * `apt`でインストール
  * ワークスペースでビルド<br />　
* 例: ウェブカメラの映像をウェブブラウザに表示<br />（次ページから）
  * USBのウェブカメラを持っている人は試してみましょう
  * カメラ付きのUbuntu 20.04の入ったノートPCでもできます

---

## aptによるROSパッケージの<br />インストール

* 手順
```
$ sudo apt install ros-noetic-cv-camera
### おそらく不要だが念のため ###
$ sudo apt install ros-noetic-cv-bridge
```
  * ROS関係のパッケージをAPTで扱う設定: `step1.bash`が`/etc/apt/sources.list.d/ros-latest.list`に書き込み

---

## コードからのビルド

* `web_video_server`というパッケージをGitHubから持ってくる
  * Ubuntu 18.04以前では`apt`でインストール可能
  * そのうちUbuntu 20.04でも可能になるはず

```
$ sudo apt install ffmpeg （いらないかもしれません）
$ cd ~/catkin_ws/src/
$ git clone https://github.com/GT-RAIL/async_web_server_cpp.git
$ git clone https://github.com/RobotWebTools/web_video_server.git
$ ( cd ~/catkin_ws/ && catkin_make -j 4 )
### ↑ -j 4はCPUを4つ使うというパラメータ（非力なラズパイでは避ける） ###
```


---

## 動作確認

* `roscore`と`rosrun`でプログラムを立ち上げ
  * ノード（後述）の通信を仲介する`roscore`を立ち上げ
      ```
      $ roscore &   ←「&」をつけてバックグラウンドで起動すると端末数が節約できる
      ```
  * カメラを立ち上げ
      ```
      $ ls /dev/video*
      /dev/video0     ←ビデオのデバイスファイルがあるか確認
      $ rosrun cv_camera cv_camera_node
      ```
  * サーバを立ち上げ
      ```
      $ rosrun web_video_server web_video_server
      [ INFO] [1605588925.417323132]: Waiting For connections on 0.0.0.0:8080
      ```
* ブラウザで`http://ラズパイのIPアドレス:8080`にアクセスするとカメラの映像が確認できる


---

## ROSノード

* ROS上で動くプログラムは 「ノード」と呼ばれる
  * Unixで言う「プロセス」の言い換え
  * 前のスライドの`cv_camera_node`と`web_video_server` <br />　
* ノードは連携して動く
  * `cv_camera_node`: カメラ画像をROS、OpenCV用の形式へ変換
  * `web_video_server`: 変換されたデータをもらってウェブ配信

```
### ディレクトリのように管理されている ###
$ rosnode list
/cv_camera           ←cv_camera_nodeのノード
/rosout
/web_video_server    ←web_video_serverのノード
```

---

## ROSトピックとメッセージ

* 今度はrostopic listと打ってみる
  * データをやり取りする口（トピック）が表示される
```
$ rostopic list
/cv_camera/camera_info
/cv_camera/image_raw
（略）
```
* トピックからデータを取り出す
  * このデータは「メッセージ」と呼ばれる
```
$ rostopic echo /cv_camera/image_raw | head -c 300
header:
  seq: 36218
  stamp:
    secs: 1605590255
    nsecs: 752682094
  frame_id: "camera"
height: 480
width: 640
encoding: "bgr8"
is_bigendian: 0
step: 1920
data: [255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 2・・・
```

---

## ノードとトピックの関係

* 各ノードがトピックを通じてメッセージを融通することで全体として仕事を行う
  * ノードは、いくつかの「パブリッシャ」と「サブスクライバ」を持つ
    * データを出す側が**パブリッシャ**
    * データを受け取る側が**サブスクライバ**<br />　
* この構造でノードの柔軟な組み換えが可能に
  * インタフェースが同じならプログラムを取り替えられる
  * 例: mjpeg_server（古い） -> web_video_server（新しい）

---

## ROSパッケージの構成

* `web_video_server`の中身を見てみましょう
```
ubuntu@ubuntu:~/catkin_ws/src/web_video_server$ l
AUTHORS.md     CMakeLists.txt  README.md  mainpage.dox  src/
CHANGELOG.rst  LICENSE         include/   package.xml
```
  * `package.xml`: パッケージマニフェスト
  * `CMakeLists.txt`: CMakeのスクリプト
  * コード
    * `src`: ソースコード（.cppファイル）
    * `include`: ヘッダファイル（.hファイル）

単にコードがあるだけではなく、他人やシステムが使うための情報が整備されている

---

## ROSプログラミング

* パッケージを作る
* ノードを作る
* パブリッシャ、サブスクライバを作る

---

## パッケージを作る

* `catkin_create_pkg`で作成
  * 作成するパッケージ名、使用するライブラリを指定
```bash
$ cd ~/catkin_ws/src
$ catkin_create_pkg mypkg rospy
 Created file mypkg/package.xml
 Created file mypkg/CMakeLists.txt
 Created folder mypkg/src
 Successfully created files in /home/ubuntu/catkin_ws/src/mypkg. （略）
```
* パッケージ内に`scripts`というディレクトリを作成
  * ここにノードとなるプログラムを置く
```bash
$ cd mypkg/
$ mkdir scripts
$ cd scripts/
```

---

## ノードの作成1

* パブリッシャを1個持つノード`count.py`（下のコード）を記述
  * `rospy.Publisher`を作って定期的にデータを投げる
    * `count_up`というトピック
    * 型はInt32（メッセージには型がある）
<div style="font-size:70%">


```python
#!/usr/bin/env python3
import rospy
from std_msgs.msg import Int32

rospy.init_node('count')                                # ノード名「count」に設定
pub = rospy.Publisher('count_up', Int32, queue_size=1)  # パブリッシャ「count_up」を作成
rate = rospy.Rate(10)                                   # 10Hzで実行
n = 0
while not rospy.is_shutdown():
    n += 1
    pub.publish(n)
    rate.sleep()
```

</div>

---

## ノードの実行

```bash
端末1$ roscore
端末2$ chmod +x count.py    ←実行できるようにパーミッション設定
端末2$ rosrun mypkg count.py
```

* 動作確認
  * `rosnode list`と`rostopic list`でノードとトピックの確認を
  * `rostopic echo`で`count_up`からデータを取り出してみましょう

```bash
端末3$ rosnode list
/count
/rosout
端末3$ rostopic list
/count_up
/rosout
/rosout_agg
端末3$ rostopic echo /count_up 
data: 1430
---
data: 1431
---
...
```

---

## ノードの作成2

* サブスクライバ1個を持つノード`twice.py`（下のコード）を記述
* `rospy.Subscriber`を使う
  * count_upという名前のトピックを購読する
  * 型はInt32
  * データを受け取ったときにcbという関数で処理
    * コールバック関数

<div style="font-size:70%">

```python
#!/usr/bin/env python3
import rospy
from std_msgs.msg import Int32

def cb(message):
    rospy.loginfo(message.data*2)

if __name__ == '__main__':
    rospy.init_node('twice')
    sub = rospy.Subscriber('count_up', Int32, cb)
    rospy.spin()
```

</div>

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

## ノードの作成3

* データを受け取って加工するノード
  * `twice.py`にパブリッシャを追加

<div style="font-size:70%">

```python
#!/usr/bin/env python3
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

</dev>

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

## その他

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

## 参考文献

* [小倉: ROSではじめるロボットプログラミング, 工学社, 2015.](https://www.kohgakusha.co.jp/books/detail/978-4-7775-1901-9)
* [上田: Raspberry Piで学ぶ　ROSロボット入門, 日経BP, 2017.](http://ec.nikkeibp.co.jp/item/books/261040.html)

