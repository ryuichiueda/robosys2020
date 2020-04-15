# ロボットシステム学第13回

上田 隆一

2019年12月20日@千葉工業大学

## 今日の内容

* ROSの続き
  * roslaunch
  * rosbridge
    * ROS+ウェブプログラミング
      * ウェブプログラミングの経験がなくても一度やって免疫をつける目的

### roslaunch

* いちいちノードを手でたくさん立ち上げるのは面倒
* XMLに立ち上げに関する情報を置いて、ひとつのコマンドで立ち上げられる仕組み

```bash
$ cd ~/catkin_ws/src/mypkg/      #roscd mypkgでもよい
$ mkdir launch
$ cd launch
```

* `launch`ディレクトリの下で次のように記述
  * ファイル名は`mypkg.launch`としておきましょう

```mypkg.launch
<launch>
  <node pkg="mypkg" name="count" type="count.py" required="true" />
  <node pkg="mypkg" name="twice" type="twice.py" required="true" />
</launch>
```


* 実行

```bash
$ roslaunch mypkg mypkg.launch 
... logging to /home/ubuntu/.ros/log/2a1397ac-d0df-11e5-88c9-b827eb62a884/roslaunch-ubuntu-2217.log
Checking log directory for disk usage. This may take awhile.
Press Ctrl-C to interrupt
Done checking log file disk usage. Usage is <1GB.

started roslaunch server http://localhost:45489/

SUMMARY
========

PARAMETERS
 * /rosdistro: kinetic
 * /rosversion: 1.12.7

NODES
  /
    count (mypkg/count.py)
    twice (mypkg/twice.py)

ROS_MASTER_URI=http://localhost:11311

core service [/rosout] found
process[count-1]: started with pid [2235]
process[twice-2]: started with pid [2236]
（立ち上がりっぱなしになる）
```

* 通信を一通り試したら切りましょう。

## rosbridge

* ROSからウェブにデータを飛ばしてみましょう。
  * rosbridgeというソフトウェア群を使います
  * http://wiki.ros.org/rosbridge_suite
* ウェブプログラミングの経験がなくても手を動かす

### インストール


```bash
$ sudo apt install ros-kinetic-rosbridge-suite
```

* うまくいかないときは`sudo apt update`してからインストール
### HTMLファイルの準備 * scriptsの中にindex.htmlを準備

```index.html
test
```


### ウェブサーバの準備

* Pythonの簡易ウェブサーバを使います

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import rospy, os
import SimpleHTTPServer

def kill():
    os.system("kill -KILL " + str(os.getpid())) #強制シャットダウン

os.chdir(os.path.dirname(__file__))  #scriptsディレクトリが見えるように
rospy.init_node("webserver")         #rosのノード登録
rospy.on_shutdown(kill)              #kill関数の登録
SimpleHTTPServer.test()              #サーバ立ち上げ
```

### launchファイルへの登録

* mypkg.launchにwebserverノードを登録します

```mypkg.launch
<launch>
  <node pkg="mypkg" name="count" type="count.py" required="true" />
  <node pkg="mypkg" name="twice" type="twice.py" required="true" />
  <node pkg="mypkg" name="webserver" type="webserver.py" args="8000" required="true" />  <-これを追加
</launch>
```

### 立ち上げ

```bash
$ roslaunch mypkg mypkg.launch 
```

* この後、手元のPCのブラウザで`http://ラズパイのIPアドレス:8000`を閲覧
  * 次のように見えたらOK

<img style="width:50%" src="./web.png" />


### HTMLの記述

* 最小限のHTMLを書きます
  * span...のところに`count_up`トピックの値を出すように記述
  * rosbridgeを使うためのjavascriptファイルをダウンロードできるように指定

```index.html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8">
  </head>
  <body>
   <span id="count">not received</span>

<!-- 本来はコメントアウトされているリンクが正式なものであるが、12/8現在、リンク切れ -->
<!--<script src="http://cdn.robotwebtools.org/EventEmitter2/current/eventemitter2.min.js"></script> -->
   <script src="https://static.robotwebtools.org/EventEmitter2/current/eventemitter2.min.js"></script>
<!--   <script src="http://cdn.robotwebtools.org/roslibjs/current/roslib.min.js"></script> -->
   <script src="https://static.robotwebtools.org/roslibjs/current/roslib.min.js"></script>
   <script src="./main.js"></script>
  </body>
</html>
```

* roslaunchを実行し、ブラウザで「not received」と出ることを確認

### JavaScriptの記述

* 書いたことのない人もとにかく書きましょう。

```javascript
var ros = new ROSLIB.Ros({ url : 'ws://' + location.hostname + ':9000' });
                                                   
ros.on('connection', function() {console.log('websocket: connected');});
ros.on('error', function(error) {console.log('websocket error: ', error); });
ros.on('close', function() {console.log('websocket: closed');});

var ls = new ROSLIB.Topic({
        ros : ros,
        name : '/twice',
        messageType : 'std_msgs/Int32'
});

ls.subscribe(function(message) {
        str = JSON.stringify(message);
        document.getElementById("count").innerHTML = str;
        console.log(str);                                  
});
```

### launchファイルの編集

次のようにrosbridge_serverというパッケージのlaunchファイルを探して立ち上げるための記述を追加。

```xml
<launch>
  <include file="$(find rosbridge_server)/launch/rosbridge_websocket.launch">
     <arg name="port" value="9000"/>
  </include>
  （以下略）
```

### 実行

* ブラウザで閲覧

<img style="width:50%" src="./web_topic.png" />

