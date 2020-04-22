# ロボットシステム学第12回

上田 隆一

千葉工業大学

---

## 今日の内容

* ROS2
* 参考図書
    * 近藤 豊: [ROS2ではじめよう 次世代ロボットプログラミング](https://gihyo.jp/book/2019/978-4-297-10742-0), 技術評論社, 2019. 

---

## ROS2

* ROS（ROS1）の次のバージョン（互換性なし）
* なぜ開発されたか
    * ROSが普及して使用される場面が増加<br />
    $\rightarrow$当初想定していなかった場面も増加
        * セキュリティー、製品化、シビアな通信環境、・・・

アーキテクチャから作り直し

---

## ROS2のインストール

* 難しい話は後回しにしてインストール
    * まずは使ってみましょう<br />　
* [インストールスクリプト](https://github.com/ryuichiueda/ros2_setup_scripts_Ubuntu18.04_desktop/blob/master/setup.bash)
    * 手順は次の通り

```bash
$ git clone https://github.com/ryuichiueda/ros2_setup_scripts_Ubuntu18.04_desktop.git
$ cd ros2_setup_scripts_Ubuntu18.04_desktop
$ ./setup.bash
$ source ~/.bashrc
```

---

## 動作確認

roscoreは不要

* パブリッシャを持つサンプルノードの起動
```bash
端末1$ ros2 run demo_nodes_py talker
・・・
[INFO] [talker]: Publishing: "Hello World: 61"
[INFO] [talker]: Publishing: "Hello World: 62"
・・・
```
* リスナーを持つサンプルノードの起動
```bash
端末2$ ros2 run demo_nodes_py listener
・・・
[INFO] [listener]: I heard: [Hello World: 55]
[INFO] [listener]: I heard: [Hello World: 56]
・・・
```

---

## ROS2のパッケージを作る

Pythonのものを作ってみましょう（[参考](https://index.ros.org/doc/ros2/Tutorials/Developing-a-ROS-2-Package/#python-packages)）

* これからやること
    * ワークスペースを作る 
    * 初期状態のパッケージを作る
    * パッケージ情報の記述
    * パッケージのビルド
    * パブリッシャの実装
    * サブスクライバの実装


---

## ワークスペースの作成

* とりあえずディレクトリを作るだけ

```bash
$ mkdir -p ros2_ws/src
$ tree ros2_ws/
ros2_ws/
└── src
1 directory, 0 files
```

---

## 初期状態のパッケージを作る

* 空のパッケージを一つ作る
    * `ros2 pkg create`を使う
    * 下の例: mypkgというパッケージを作成

```bash
$ cd ~/ros2_ws/src/
$ ros2 pkg create mypkg --build-type ament_python
going to create a new package
package name: mypkg
・・・
creating ./mypkg/test/test_pep257.py
$ tree
.
└── mypkg
    ├── mypkg
    │   └── __init__.py
    ├── package.xml
    ├── resource
    │   └── mypkg
    ├── setup.cfg
    ├── setup.py
    └── test
        ├── test_copyright.py
        ├── test_flake8.py
        └── test_pep257.py
4 directories, 8 files
```

---

## パッケージ情報の記述1

* `package.xml`を編集する
    * パッケージマニフェストというもの
    * パッケージの情報を記述したファイル
        * 説明（description）、メンテナ、ライセンスをちゃんと書く

```
$ head package.xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/..."?>
<package format="3">
  <name>mypkg</name>
  <version>0.0.0</version>
  <description>a package for practice</description>
  <maintainer email="ryuichiueda@example.com">Ryuichi Ueda</maintainer>
  <license>BSD</license>

  <test_depend>ament_copyright</test_depend>
・・・
```

---

## パッケージ情報の記述2

* `setup.py`の編集
    * Pythonのモジュールで使われるインストールスクリプト
        * 変更箇所は`package.xml`と同じ

```
$ cat setup.py
from setuptools import setup

package_name = 'mypkg'

setup(
    name=package_name,
    version='0.0.0',
    ・・・
    maintainer='Ryuichi Ueda',
    maintainer_email='ryuichiueda@gmail.com',
    description='a package for practice',
    license='BSD',
・・・
```

---

## パッケージのビルド

```bash
### ビルド ###
$ cd ~/ros2_ws/
$ colcon build
Starting >>> mypkg
Finished <<< mypkg [2.46s]

Summary: 1 package finished [2.97s]
### パスを通す ###
$ source install/setup.bash
$ source install/local_setup.bash
### パスが通っているか確認 ###
$ ros2 pkg list | grep mypkg
mypkg

```

---

## パブリッシャの作成

