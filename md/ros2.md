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


```bash
端末1$ ros2 run demo_nodes_py talker
```

